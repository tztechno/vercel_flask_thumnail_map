```
2024-09-05 18:00
後半のapp処理をgasでtriggerさせる
app,index,gasの連携
元々はlocalで地図+写真の静的サイトを作る予定だったがそれは無理ということですね







2024-09-05 12:00
flask app
templates/index.html上にあるボタンを押すと
GASが起動して、スプレッドシートにimg_pathとthumbnail_pathが作られる
app.pyでは、img_pathのimageからEXif情報（lat,long,time）情報を取得する
それを使って、static/map_data.htmlを生成する
mapの中に撮影位置にcameraアイコンがあり、クリックするとリンク先のthumbnail画像が開く
修正前のapp.pyを示す


```