# Full pipeline

After conducting accurate annotations published in [Nuclei](https://www.sciencedirect.com/science/article/pii/S2352340922009726) and [Cytosol](https://www.sciencedirect.com/science/article/pii/S2352340924011107),
training U-Net and HoVer-Net models for segmenting the nuclei channel, we have compared the top 3 best models from each architecture and at the end we have used the best U-Net model that we have trained on
our own annotated images as well as some images from [BBBC](https://bbbc.broadinstitute.org/) image set.
The model's performance on test set were validated where it had 85% F1-score on 90% overlapping threshold and 88% average Jaccard index.


The experiment were conducted in two Oxidative stress (A, B) and non-oxiditive stress (C,D) groups each group has one repetition all saved in 79 groups for almost 18000 genes.
The status of each group of raw data and how we proceeded to analyse is summarized in progress_log/plateProgress_done.xlsx file.
Some of plates were missing or there were several copies. We had to deal with missing or repeated data.


The overall process for each plate is as follows:
1)	Download from swestore
2)	Run extract_conversion.py script over plates to extract only d0 channel.
3)	Normalize them to 8bit images using bash command
4)	Save the names in 4_filelist folder  and remove .C01 from  the folder and only keep .png ones (6144 images in each full plate) 
5)	Copy  and run prediction model over them
6)	Run area_size.py script over them and copy the files to A, or B, or C, or D 
7)	Run the plot and visualization scripts over all of them




## Download and Extract

For downloading plates from Swestore, we have used:


```bash
lftp https://username@webdav.swestore.se/snic/folder/
```

For downloading whole plate we used:
```bash
get plate_number1.tar.gz plate_number2.tar.gz ...
```

For extracting and converting the format of images we have used "bfconvert" function of [bftools](https://docs.openmicroscopy.org/bio-formats/5.7.1/users/comlinetools/index.html) command line tool. 

This is conducted through command line or  preprocessing/extract_conversion.py script and subprocess library.


The following command should run over images after bfconvert command to normalize images to 8-bit format,

```bash
ls *.png ; while read file; do convert file -auto-level  -depth 8 -define quantum:format=unsigned -type grayscale file; done
```

or through extract_conversion.py script.

Besides, We extracted multi plates through :

```bash
for FILE in *.tar.gz; do
    echo ${FILE} | cut -d '/' -f 3
    sbatch -A  project_name -n 1 -t 5:00:00 --wrap="python extract_conversion.py  $(echo ${FILE}|cut -d '/' -f 3)" 
    sleep 1 # pause to be kind to the scheduler
done

```


## Image list


```bash
ls  *.png > ../4_filelists/plate_num_names.txt
```


## Run Prediction script

For running prediction script we have used CPU multi-thread parallel prediction and gpu by adding following parts to the code.

```python
global model
global graph

from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor, as_completed
from multiprocessing import cpu_count

with open(path_files_list) as image_list:
    image_names_all = [os.path.join(path_files + sys.argv[1]+'/', f.strip()) for f in image_list]



clear_session()

model = utils.model_builder.get_model_3_class(1104, 1104,1)                                             
model.load_weights('model_14.hdf5')  ## loading best model here from UNet models
model._make_predict_function()
tf.Graph()
graph = tf.get_default_graph()
test = [prediction_images(image_names_all[i_batch*128:(i_batch+1)*128]) for i_batch in range(48)]  ## use this if you do not want multi-threading


## use this if you want multi-threading
CPU_LIMIT=128  ## This is the cluster limit
with ThreadPoolExecutor(min(CPU_LIMIT,cpu_count())) as executor:
    print('befoooooooooooooooooore submit function')
    futures=[executor.submit(prediction_images ,image_names_all[i_batch*16:(i_batch+1)*16]) for i_batch in range(384)]#range(len(image_names_all))] 

    for future in as_completed(futures):
        res=future.result()
        print('resssssssssssssuuuuuuuuuuuuuult: ',res)
 

```

We have also predict several plates using only gpu by

```bash
python run_prediction.py
```

where, we have 

```python
import subprocess

subprocess.run("python prediction.py MFGTMPcx7_170801050001 ../data/4_filelists/MFGTMPcx7_170801050001_names.txt", shell=True)
subprocess.run("python prediction.py MFGTMPcx7_170801100001 ../data/4_filelists/MFGTMPcx7_170801100001_names.txt", shell=True)
...
```

## Area and Number of Nuclei

After running prediction script, for every plate, a "segm" folder was created where the predicted segmentation masks were stored. By running pipeline/plate_script/area_count.py

We will have a .csv file for each plate where we have three columns as follows. 



|Pred_Object   | Image_name                              |  Area  | 
|--------------|-----------------------------------------|--------|
| 0            | MFGTMPcx7_170525180001_A01f00d0.png     |   141  |
| 1            | MFGTMPcx7_170525180001_A01f00d0.png     |   1545 | 
| 2            | MFGTMPcx7_170525180001_A01f00d0.png     |   2179 | 
| -            | -                                       |   -    | 


## Visualization

This then saved and averaged for all wells for each plate and connected to the gene that were knocked-down at that well. The visualization and numercial results are all in 
screen_visualization and results directories.


