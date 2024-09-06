import os
from PIL import Image
from PIL.ExifTags import TAGS, GPSTAGS
from datetime import datetime
import csv
import folium
from statistics import mean

def resize_image(input_path, output_path, max_width):
    with Image.open(input_path) as img:
        original_width, original_height = img.size
        
        if original_width > max_width:
            ratio = max_width / original_width
            new_height = int(original_height * ratio)
            img = img.resize((max_width, new_height), Image.LANCZOS)

        img.save(output_path)

def convert_to_degrees(value):
    d, m, s = value
    return d + (m / 60.0) + (s / 3600.0)

def get_gps_coordinates(exif_data):
    lat = lon = date_time = None

    if 'GPSInfo' in exif_data:
        gps_info = {GPSTAGS.get(key, key): value for key, value in exif_data['GPSInfo'].items()}
        
        if 'GPSLatitude' in gps_info and 'GPSLongitude' in gps_info:
            lat = convert_to_degrees(gps_info['GPSLatitude'])
            lon = convert_to_degrees(gps_info['GPSLongitude'])
            
            if gps_info['GPSLatitudeRef'] != 'N':
                lat = -lat
            if gps_info['GPSLongitudeRef'] != 'E':
                lon = -lon

    if 'DateTimeOriginal' in exif_data:
        date_time_str = exif_data['DateTimeOriginal']
        try:
            date_time = datetime.strptime(date_time_str, '%Y:%m:%d %H:%M:%S')
        except ValueError:
            pass

    return lat, lon, date_time

def process_images(photos_dir, static_dir, thumbnail_size):
    if not os.path.exists(static_dir):
        os.makedirs(static_dir)

    csv_path = os.path.join(static_dir, 'image_data.csv')
    coordinates = []

    with open(csv_path, 'w', newline='') as csvfile:
        csv_writer = csv.writer(csvfile)
        csv_writer.writerow(['Filename', 'Latitude', 'Longitude', 'DateTime', 'Thumbnail'])

        for filename in os.listdir(photos_dir):
            if filename.lower().endswith(('.png', '.jpg', '.jpeg', '.gif')):
                input_path = os.path.join(photos_dir, filename)
                thumbnail_name = f'thumb_{filename}'
                output_path = os.path.join(static_dir, thumbnail_name)

                resize_image(input_path, output_path, thumbnail_size)

                with Image.open(input_path) as img:
                    exif_data = {TAGS.get(key, key): value for key, value in img._getexif().items() if key in TAGS}
                    lat, lon, date_time = get_gps_coordinates(exif_data)

                if lat and lon and date_time:
                    csv_writer.writerow([filename, lat, lon, date_time, thumbnail_name])
                    coordinates.append((lat, lon))

    return csv_path, coordinates

def create_map_html(csv_path, output_html, coordinates):
    if not coordinates:
        print("No valid coordinates found. Using default map center.")
        center_lat, center_lon = 0, 0
        zoom_start = 2
    else:
        center_lat = mean(coord[0] for coord in coordinates)
        center_lon = mean(coord[1] for coord in coordinates)
        
        # Calculate appropriate zoom level
        lat_range = max(coord[0] for coord in coordinates) - min(coord[0] for coord in coordinates)
        lon_range = max(coord[1] for coord in coordinates) - min(coord[1] for coord in coordinates)
        max_range = max(lat_range, lon_range)
        
        if max_range < 0.01:
            zoom_start = 15
        elif max_range < 0.1:
            zoom_start = 12
        elif max_range < 1:
            zoom_start = 10
        elif max_range < 5:
            zoom_start = 8
        else:
            zoom_start = 6

    m = folium.Map(location=[center_lat, center_lon], zoom_start=zoom_start)

    with open(csv_path, 'r') as csvfile:
        csv_reader = csv.reader(csvfile)
        next(csv_reader)  # Skip header
        for row in csv_reader:
            filename, lat, lon, date_time, thumbnail = row
            lat, lon = float(lat), float(lon)
            
            popup_html = f"""
            <img src='static/{thumbnail}' width='150px'><br>
            Filename: {filename}<br>
            Date/Time: {date_time}
            """
            
            folium.Marker(
                location=[lat, lon],
                popup=folium.Popup(popup_html, max_width=200),
                icon=folium.Icon(icon='camera', prefix='fa')
            ).add_to(m)

    m.save(output_html)

def main():
    photos_dir = 'photos'
    static_dir = 'static'
    thumbnail_size = 200
    output_html = 'index.html'

    csv_path, coordinates = process_images(photos_dir, static_dir, thumbnail_size)
    create_map_html(csv_path, output_html, coordinates)

if __name__ == "__main__":
    main()