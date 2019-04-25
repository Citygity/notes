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

## 1.Training script

I finetuned my model on coco dataset,download the pretrained weight first.

```python
# Directory to save logs and trained model
MODEL_DIR = os.path.join(ROOT_DIR, "logs")

# Local path to trained weights file
COCO_MODEL_PATH = os.path.join(MODEL_DIR, "mask_rcnn_coco.h5")
# Download COCO trained weights from Releases if needed
if not os.path.exists(COCO_MODEL_PATH):
    utils.download_trained_weights(COCO_MODEL_PATH)
```

Adjust the config class to fit your data.IMAGES_PER_GPU was set to 1 because my train data's size is (1920,1020).

```python
class ShapesConfig(Config):
    """Configuration for training on the toy shapes dataset.
    Derives from the base Config class and overrides values specific
    to the toy shapes dataset.
    """
    # Give the configuration a recognizable name
    NAME = "shapes" 

    # Train on 1 GPU and 1 images per GPU.  Batch size is 8 (GPUs * images/GPU).
    GPU_COUNT = 7
    IMAGES_PER_GPU = 1

    # Number of classes (including background)
    NUM_CLASSES = 1 + 3  # background + 3 shapes

    # Use small images for faster training. Set the limits of the small side
    # the large side, and that determines the image shape.
    #!!!recommend to refer to config.py for more details
    IMAGE_MIN_DIM = 512
    IMAGE_MAX_DIM = 512

    # Use smaller anchors because our image and objects are small
    RPN_ANCHOR_SCALES = (8, 16, 32, 64, 128)  # anchor side in pixels

    # Reduce training ROIs per image because the images are small and have
    # few objects. Aim to allow ROI sampling to pick 33% positive ROIs.
    TRAIN_ROIS_PER_IMAGE = 32

    # Use a small epoch since the data is simple
    # fit_generator was used to train the model, it determines the frequency of saving 		# model and validation
    # steps_per_epoch: Integer. Total number of steps (batches of samples) to yield from 	 #	generator before declaring one epoch finished and starting the next epoch. It 		# should typically be equal to ceil(num_samples / batch_size) Optional for Sequence: 	 #if unspecified, will use the len(generator) as a number of steps.
    STEPS_PER_EPOCH = 1000

    # use small validation steps since the epoch is small
    VALIDATION_STEPS = 5

config=ShapesConfig()
config.display()

```

​	

```python
class RoadSignDataset(utils.Dataset):
    images_dirs = {'0927': ['0927/0927_1', '0927/0927_2', '0927/0927_3', '0927/0927_4', '0927/0927_5', '0927/0927_6',
                            '0927/0927_7', '0927/0927_8'],\
                   '1023': ['1023/1023_1', '1023/1023_2', '1023/1023_3', '1023/1023_11', '1023/1023_12'],\
                   '1025': ['1025/1025_13', '1025/1025_14'],\
                   '1101': ['1101/1101_17', '1101/1101_18', '1101/1101_19', '1101/1101_20', '1101/1101_21'],\
                   '1107': ['1107/1107_22', '1107/1107_23', '1107/1107_24', '1107/1107_25', '1107/1107_26',
                            '1107/1107_27', '1107/1107_28']}

    def load_shapes(self, count, img_floder, mask_floder, imglist, image_root_path):
        """Generate the requested number of synthetic images.
        count: number of images to generate.
        height, width: the size of the generated images.
        """
        # Add classes, change this part to adjust your dataset
        self.add_class("road_sign", 1, "sign")
        self.add_class("road_sign", 2, "sign_back")
        self.add_class("road_sign", 3, "side_sign")
        self.add_class("road_sign", 4, "side_sign_back")
        self.add_class("road_sign", 5, "led")
        # iterate your dataset ,add your path of train image and information of mask including the category and coordinate of polygons
        for i,filename in enumerate(imglist):
            image_file_path=os.path.join(img_floder,filename)
            mask_file_path=os.path.join(mask_floder,filename[:-4]+'txt')
            with open(mask_file_path) as f:
                mask_infos = json.load(f)
                #mask_infos = list(mask_infos);
                #polygons = [content[u'points'] for content in mask_infos]
                print("load_shape:{};imageid:{}".format(image_file_path, i))
                self.add_image("road_sign", image_id=i, path=image_file_path,
                               width=1920, height=1020, mask_path=mask_file_path, mask_infos=mask_infos)
	# generate your mask by your mask_infos
    def load_mask(self, image_id):
        """Generate instance masks for shapes of the given image ID.
        """
        #change this line below to suit your dataset
        categorys={'40':1,'51':2,'59':3,'60':4,'61':5}
        info = self.image_info[image_id]
        mask = np.zeros([info["height"], info["width"], len(info["mask_infos"])],
                        dtype=np.uint8)
        labels_form=[]
        for i,child in enumerate(info["mask_infos"]):
            label_category=child['category']
            points=child['points']
            x=np.array(points)[:,0]
            y=np.array(points)[:,1]
            rr,cc=skimage.draw.polygon(y,x)
            mask[rr, cc, i] = 1
            labels_form.append(categorys[label_category])
        return mask.astype(np.bool), np.array(labels_form).astype(np.int32)



  
		
```

some other settings as below:

```python
# Training dataset
dataset_train = ShapesDataset()
dataset_train.load_shapes(500, config.IMAGE_SHAPE[0], config.IMAGE_SHAPE[1])
dataset_train.prepare()

# Validation dataset
dataset_val = ShapesDataset()
dataset_val.load_shapes(50, config.IMAGE_SHAPE[0], config.IMAGE_SHAPE[1])
dataset_val.prepare()

# Create model in training mode
model = modellib.MaskRCNN(mode="training", config=config,
                          model_dir=MODEL_DIR)

# Which weights to start with?
init_with = "coco"  # imagenet, coco, or last

if init_with == "imagenet":
    model.load_weights(model.get_imagenet_weights(), by_name=True)
elif init_with == "coco":
    # Load weights trained on MS COCO, but skip layers that
    # are different due to the different number of classes
    # See README for instructions to download the COCO weights
    model.load_weights(COCO_MODEL_PATH, by_name=True,
                       exclude=["mrcnn_class_logits", "mrcnn_bbox_fc",
                                "mrcnn_bbox", "mrcnn_mask"])
elif init_with == "last":
    # Load the last model you trained and continue training
    model.load_weights(model.find_last(), by_name=True)


# Fine tune all layers
# Passing layers="all" trains all layers. You can also
# pass a regular expression to select which layers to
# train by name pattern.
# layers: Allows selecting wich layers to train. It can be:
# - A regular expression to match layer names to train
# - One of these predefined values:
# heads: The RPN, classifier and mask heads of the network
# all: All the layers
# 3+: Train Resnet stage 3 and up
# 4+: Train Resnet stage 4 and up
# 5+: Train Resnet stage 5 and up
model.train(dataset_train, dataset_val,
            learning_rate=config.LEARNING_RATE / 10,
            epochs=2,
            layers="all")
```

some tips about hyper parameters：

## Mini Masks

Instance binary masks can get large when training with high resolution images. For example, if training with 1024x1024 image then the mask of a single instance requires 1MB of memory (Numpy uses bytes for boolean values). If an image has 100 instances then that's 100MB for the masks alone.

To improve training speed, we optimize masks by:

- We store mask pixels that are inside the object bounding box, rather than a mask of the full image. Most objects are small compared to the image size, so we save space by not storing a lot of zeros around the object.
- We resize the mask to a smaller size (e.g. 56x56). For objects that are larger than the selected size we lose a bit of accuracy. But most object annotations are not very accuracy to begin with, so this loss is negligable for most practical purposes. Thie size of the mini_mask can be set in the config class.



## config.py

used in ProposalLayer(),to determine numbers of ROIs of nms.

```python
    # ROIs kept after non-maximum suppression (training and inference)
    POST_NMS_ROIS_TRAINING = 2000
    POST_NMS_ROIS_INFERENCE = 1000
```

## Anchors

The order of anchors is important. **Use the same order in training and prediction phases.** And it must match the order of the convolution execution.

For an FPN network, the anchors must be ordered in a way that makes it easy to match anchors to the output of the convolution layers that predict anchor scores and shifts.

- Sort by pyramid level first. All anchors of the first level, then all of the second and so on. This makes it easier to separate anchors by level.
- Within each level, sort anchors by feature map processing sequence. Typically, a convolution layer processes a feature map starting from top-left and moving right row by row.
- For each feature map cell, pick any sorting order for the anchors of different ratios. Here we match the order of ratios passed to the function.

**Anchor Stride:** In the FPN architecture, feature maps at the first few layers are high resolution. For example, if the input image is 1024x1024 then the feature meap of the first layer is 256x256, which generates about 200K anchors (256*256*3). These anchors are 32x32 pixels and their stride relative to image pixels is 4 pixels, so there is a lot of overlap. We can reduce the load significantly if we generate anchors for every other cell in the feature map. A stride of 2 will cut the number of anchors by 4, for example.

In this implementation we use an anchor stride of 2, which is different from the paper.