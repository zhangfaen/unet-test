As unet_carvana_scale0.5_epoch2-by-milesial.pth is more than 100MB, it can not be pushed to github.
We zip it into 2 splited zip files,by

zip -s 60m unet_carvana_scale0.5_epoch2-by-milesial.pth.zip unet_carvana_scale0.5_epoch2-by-milesial.pth

then push. After download, we use 

zip -F unet_carvana_scale0.5_epoch2-by-milesial.pth.zip  --out all-in-one.zip

to convert 2 splited files into one. Then unzip the file by

unzip all-in-one.zip 



