# Photovoltaic-Cell-Fault-detection
To do list:
 Import model detection (SSD & YOLO3)
 Example use Trained Model
 Train and Evaluate Model with own data
 Model Panel Detection (SSD7)
 Model Panel Detection (YOLO3)
 Model Soiling Fault Detection (YOLO3)
 Model Diode Fault Detection (YOLO3)
 Model Other Fault Detection
 Model Fault Panel Disconnect
 
 Type of Data
The images used for the design of this model were extracted by air analysis, specifically: FLIR aerial radiometric thermal infrared pictures, taken by UAV (R-JPEG format). Which were converted into .jpg images for the training of these detection models. Example FLIR image:

FLIR

Same image in .jpg format:

JPG

Training
1. Data preparation
View folder Train&Test_A/ and Train&Test_S/, example of panel anns and soiling fault anns.

Organize the dataset into 4 folders:

train_image_folder <= the folder that contains the train images.

train_annot_folder <= the folder that contains the train annotations in VOC format.

valid_image_folder <= the folder that contains the validation images.

valid_annot_folder <= the folder that contains the validation annotations in VOC format.

There is a one-to-one correspondence by file name between images and annotations. For create own data set use LabelImg code from : https://github.com/tzutalin/labelImg

2. Edit the configuration file
The configuration file for YOLO3 is a json file, which looks like this (example soiling fault ):

{
    "model" : {
        "min_input_size":       400,
        "max_input_size":       400,
        "anchors":              [5,7, 10,14, 15, 15, 26,32, 45,119, 54,18, 94,59, 109,183, 200,21],
        "labels":               ["1"],
	"backend": 		"full_yolo_backend.h5"
    },

    "train": {
        "train_image_folder":   "../Train&Test_S/Train/images/",
        "train_annot_folder":   "../Train&Test_S/Train/anns/",
	"cache_name":           "../Experimento_fault_1/Resultados_yolo3/full_yolo/experimento_fault_1_gpu.pkl",

        "train_times":          1,

        "batch_size":           2,
        "learning_rate":        1e-4,
        "nb_epochs":            200,
        "warmup_epochs":        15,
        "ignore_thresh":        0.5,
        "gpus":                 "0,1",

	"grid_scales":          [1,1,1],
        "obj_scale":            5,
        "noobj_scale":          1,
        "xywh_scale":           1,
        "class_scale":          1,

	"tensorboard_dir":      "log_experimento_fault_gpu",
	"saved_weights_name":   "../Experimento_fault_1/Resultados_yolo3/full_yolo/experimento_yolo3_full_fault.h5",
        "debug":                true
    },

    "valid": {
        "valid_image_folder":   "../Train&Test_S/Test/images/",
        "valid_annot_folder":   "../Train&Test_S/Test/anns/",
        "cache_name":           "../Experimento_fault_1/Resultados_yolo3/full_yolo/val_fault_1.pkl",

        "valid_times":          1
    },
   "test": {
        "test_image_folder":   "../Train&Test_S/Test/images/",
        "test_annot_folder":   "../Train&Test_S/Test/anns/",
        "cache_name":          "../Experimento_fault_1/Resultados_yolo3/full_yolo/test_fault_1.pkl",

        "test_times":          1
    }
}
The configuration file for SSD300 is a json file, which looks like this (example soiling fault ) and .txt with name of images (train.txt):

{
    "model" : {
        "backend":      "ssd300",
        "input":        400,
        "labels":               ["1"]
    },

    "train": {
        "train_image_folder":   "Train&Test_S/Train/images",
        "train_annot_folder":   "Train&Test_S/Train/anns",
        "train_image_set_filename": "Train&Test_S/Train/train.txt",

        "train_times":          1,
        "batch_size":           12,
        "learning_rate":        1e-4,
        "warmup_epochs":        3,
        "nb_epochs":            100,
	       "saved_weights_name":     "Result_ssd300_fault_1/experimento_ssd300_fault_1.h5",
        "debug":                true
    },
    "valid": {
            "valid_image_folder":   "../Train&Test_D/Test/images/",
            "valid_annot_folder":   "../Train&Test_D/Test/anns/",
            "valid_image_set_filename":   "../Train&Test_D/Test/test.txt"
        },

"test": {
        "test_image_folder":   "Train&Test_S/Test/images",
        "test_annot_folder":   "Train&Test_S/Test/anns",
        "test_image_set_filename":   "Train&Test_S/Test/test.txt"
    }
}
3. Start the training process
python train_ssd.py -c config.json -o /path/to/result

or python train_ssd.py -c config.json -o /path/to/result

By the end of this process, the code will write the weights of the best model to file best_weights.h5 (or whatever name specified in the setting "saved_weights_name" in the config.json file). The training process stops when the loss on the validation set is not improved in 20 consecutive epoches.

4. Perform detection using trained weights on image, set of images
python predict_ssd.py -c config.json -i /path/to/image/or/video -o /path/output/result or python predict_yolo.py -c config.json -i /path/to/image/or/video -o /path/output/result

It carries out detection on the image and write the image with detected bounding boxes to the same folder.

Evaluation
The evaluation is integrated into the training process, if you want to do the independent evaluation you must go to the folder ssd_keras-master or keras-yolo3-master and use the following code

python evaluate.py -c config.json Example: python keras-yolo3-master/evaluate.py -c config_full_yolo_fault_1_infer.json

Compute the mAP performance of the model defined in saved_weights_name on the validation dataset defined in valid_image_folder and valid_annot_folder.
