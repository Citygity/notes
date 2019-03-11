# How to train Mask R-CNN on your own dataset

## 0.prepare your dataset

 Label your dataset with labelme or some other tools, you may inference some blogs to learn how to use it.

Below I'd list a example of my annotation file.

```json
[{"category":"59","type":"polygon","points":[[891,693],[946,690],[944,623],[888,624]]},{"category":"59","type":"polygon","points":[[1113,681],[1144,678],[1141,635],[1108,638]]},{"category":"40","type":"polygon","points":[[1044,612],[1045,613],[1046,615],[1048,616],[1049,616],[1051,617],[1053,616],[1054,616],[1055,616],[1057,615],[1059,614],[1060,612],[1060,611],[1061,608],[1061,607],[1060,604],[1058,602],[1057,600],[1054,599],[1052,599],[1049,600],[1047,601],[1045,603],[1044,605],[1043,608],[1044,610],[1044,611]]},{"category":"40","type":"polygon","points":[[1074,610],[1074,611],[1075,611],[1077,612],[1078,613],[1079,613],[1081,613],[1083,613],[1084,613],[1085,612],[1088,611],[1088,609],[1089,607],[1089,605],[1089,602],[1089,600],[1088,599],[1086,597],[1082,596],[1079,596],[1076,597],[1074,599],[1073,601],[1072,603],[1072,605],[1073,608]]},{"category":"40","type":"polygon","points":[[1097,607],[1098,608],[1099,610],[1100,611],[1102,612],[1104,612],[1106,612],[1109,611],[1110,611],[1112,610],[1114,607],[1114,604],[1114,601],[1113,599],[1111,596],[1108,595],[1103,595],[1100,596],[1097,599],[1097,602],[1096,604]]}]
```

Create 2 folders JPEGImages and Segmentations to  separately put your training images and label files. note that ,you need to make sure the name of your lebels and training images match are corresponding one by one for the convenience of training  . Iterate those two folders to get the list of images and labels.

Then separate your dataset into training and val dataset.I did this job as below:



```python
def train_val_txt_split(images_dir,train_filenames,val_filenames,rate):
    #images_dir is a file contains all my images
    with open(images_dir,'r') as images,open(train_filenames,'w+') as train,open(val_filenames,'w+') as val:
        content=images.read()#get all content in the file
        content=content.split('\n')#Seperate the file by '\n' to get a list
        random.shuffle(content)#shuffle the list to make the list random
        train_filenames=content[:int(len(content)*rate)]#get your traindata
        train_filenames='\n'.join(train_filenames)#join the list with '\n' to get a string again
        val_filenames=content[int(len(content)*rate):]
        val_filenames='\n'.join(val_filenames)
        train.write(train_filenames)#save the file
        val.write(val_filenames)
```

