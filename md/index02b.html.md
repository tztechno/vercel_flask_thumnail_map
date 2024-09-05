<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Photo Map</title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
</head>

<body>
    <h1>Photo Map v3</h1>
    <p></p>
    <script>
            var doget_path = 'https://script.google.com/macros/s/AKfycbzr0qWTO8_Zf9Cq759_cOi2kXDXuF8clkhpoyVDuPFVegzz5NkJrIwDGi7NeDnnH4D-/exec'

            var write_path = `${doget_path}?action=summarize`;

            var flask_path = `${doget_path}?action=flask`;

            function writePaths() {
                window.open(write_path, '_blank');
            }

            function flaskPaths() {
                fetch(flask_path)
                    .then(response => response.text())
                    .then(data => {
                        document.getElementById('result').textContent = data;
                        if (data.includes('successfully')) {
                            document.getElementById('view-map').style.display = 'inline';
                        }
                    })
                    .catch(error => {
                        console.error('Error:', error);
                        document.getElementById('result').textContent = 'エラーが発生しました。';
                    });
            }

        function getAndProcessImageData() {
            fetch(display_map)
                .then(response => response.json())
                .then(data => {
                    // 取得したデータをFlaskサーバーに送信
                    fetch('/process_image', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify({ image_paths: data.map(item => item.path) }),
                    })
                        .then(response => response.json())
                        .then(result => {
                            document.getElementById('result').textContent = result.message;
                            if (result.map_created) {
                                document.getElementById('view-map').style.display = 'inline';
                            }
                        })
                        .catch(error => {
                            console.error('Error:', error);
                            document.getElementById('result').textContent = 'エラーが発生しました。';
                        });
                })
                .catch(error => {
                    console.error('Error:', error);
                    document.getElementById('result').textContent = 'データの取得に失敗しました。';
                });
        }
    </script>

    <button onclick="writePaths()">Create Thumbnails</button>
    <button onclick="flaskPaths()">Flask Go</button>

    <br><br>
    <div id="result"></div>
    <a id="view-map" href="/map" style="display: none;">View Map</a>

    {% if map_created %}
    <p>Map created successfully! <a href="/map">View Map</a></p>
    {% endif %}
</body>
</html>