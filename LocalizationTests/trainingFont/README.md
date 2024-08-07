# <b>Running Tesseract Training Tools Image</b>
Before running the image, check the Dockerfile to ensure you have the same number of threads, or if different, change the values "12" to your
exact number of threads. These values can be find in the Dockerfile in the "#Bulding tesseract" section.

### Pre-work 
Copy desired files to _sharedFolder_ in order to work with them inside container, and keep persistency so they don't get deleted when container shutsdown.

Moreover, copy desired fonts inside _fonts_ folder, so you can use them in the container. There is an Apex font as example.

***
<details open>
  <summary><h2><u>1. Start Image and Run</u></h2></summary>

  To run the container and docker image simply run Docker Desktop and type in a terminal inside _./training_ folder:

  ```
  docker compose up -d
  ```

  To get inside container just simply type:

  ```
  .\connect.bat tesseract-cont
  ```

  A short bat file that executes ``` docker exec -it tesseract-cont bash```


  **NOTE: It could take around 10 min to build the image since it has to download and clone necessary dependencies and repositories.**
</details>

<!--- 
It clones everything at the same time so you can check if has finished using ```git status```  inside each repo folder in tesseract_repos.

- <b>tesseract</b> should show next message:

  <font color="red">HEAD detached at</font> 5.2.0 

- <b>tesstrain</b> should show next message:

  <font color="red">HEAD detached at</font> 43ff100 

- <b>langdata_lstm</b> should show next message:

  On branch main. Your branch is up to date with 'origin/main'.

  nothing to commit, working tree clean

- <b>tessdata_best</b> should show next message:

  On branch main. Your branch is up to date with 'origin/main'.

  nothing to commit, working tree clean

Otherwise, wait until those messages show up.
--->
***

<details>
<summary><h2><u>2. Creating Ground Truth</u></summary>

Before training you'll have to create a database. The script ground_truth_exec.py inside _trainingFont_ is able to create a database from a plain text file, containing all characters from a certain lenguage. 

You can use also a custom file so the model is trained with your data. However make sure that your document has an extension like
[lenguage].training_text and there can not be empty lines. 

To launch the script  you need to specify, at least, a lenguage and font name, otherwise it won't work.

Type the following command:

```
python ground_truth_exec.py -l [lenguage] -f [fontName]
```

### Script Flags
Here a summary of all the flags available in the script.

| Flag |  Shortcut  | Description |
|-----------|-----------|-----------|
| --leanguage   | -l   | Lenguage you wish to train your font. Need to be recognized by tesseract.   |
| --fontname   | -f   | Font name registered in font file.|
| --directory   | -dir   | Custom directory where is stored custom text file to create ground truth.|
| --clear   | -cl   | Clear folder that stores ground truth. Need to specify also lenguage and font name.|
| --limit   | -lm   | Limit number of lines from custom text. If not specified, by default it would be 100. If the value is -1, it will be unlimited.|

Moreover, you can launch the script with ```--help``` to see the full list of flags.

**Note**: all lenguages recognized by tesseract: https://github.com/tesseract-ocr/langdata_lstm

</details>

***
<details>
<summary><h2><u>3. Training</u></summary></h2>

If you haven't placed your font inside _fonts_ folder before creating this image, copy it in mentioned folder so you can use it inside container.


In the container, copy custom font file inside _/usr/local/share/fonts_ and run the following command so the OS recognize the font.
```
fc-cache -f -v
```

Launch trainTess_exec.py script  inside _trainingFont_ folder with following syntax:

``` 
python trainTess_exec.py -l [lenguaje] -f [fontName] -it [num max training iterations]
```

For instance, to train the example font Apex with an english lenguage, it should look like this:

``` 
python trainTess_exec.py -l eng -f Apex -it 1000
```

### Script Flags
Here a summary of all the flags available in the script.

| Flag |  Shortcut  | Description |
|-----------|-----------|-----------|
| --leanguage   | -l   | Lenguage you wish to train your font. Need to be recognized by tesseract.   |
| --fontname   | -f   | Font name registered in font file.|
| --iterations   | -it   | Number of max iterations for training.|
| --clear   | -cl   | Clear folder that stores training data. Need to specify also lenguage and font name.|

Moreover, you can launch the script with ```--help``` to see the full list of flags.

**Note**: all lenguages recognized by tesseract: https://github.com/tesseract-ocr/langdata_lstm

**NOTE: The final trained model should be copied to _trainedModel_ folder inside _trainingFont_.**

<!---Copy desired lenguage traineddata to tesseract/tessdata/

Create ground-truth for desired custom font using python script.

Go to tesstrain and run with custom font and number of iterations (i.e we use Apex name font):

```
TESSDATA_PREFIX=../tesseract/tessdata make training MODEL_NAME=Apex START_MODEL=eng TESSDATA=../tesseract/tessdata MAX_ITERATIONS=100
```-->

If you get an error saying ***<span style="color:red;">bc: command not found</span>*** just run ```apt-get install bc``` and try again. 

### Testing Model

To test the model just follow the this command syntax in a terminal inside _tesstrain_ folder: 

```
tesseract [imagePath] [output]
```

For example:

```
tesseract data/Apex_data/Apex-ground-truth/eng/eng_1.tif stdout --tessdata-dir /home/tesseract_repos/tesstrain/data/Apex_data/Apex-eng-output --user-words /home/tesseract_repos/langdata_lstm/eng --psm 7 -l Apex --loglevel ALL
```

This command will extract the text from _eng_1.tif_ image and will print it in the terminal. It will use the trained data store in _Apex_data/Apex-eng-output_. 

If you wish to check all script options, you can run ```tesseract --help-extra``` in same folder, or check out its manual in https://github.com/tesseract-ocr/tesseract/blob/main/doc/tesseract.1.asc or https://muthu.co/all-tesseract-ocr-options/.

</details>

***

<details>
<summary><h2><u>4. Stop image and destroy</u></summary></h2>

Then to stop and delete container simply type:

```
docker compose down
```

If you wish to delete also the image, add ```--rmi 'all'``` to the command.

However if you just want to stop the container then run:

```
docker compose stop
```

And then, to run it again type:

```
docker compose start
```
</details>

***

## <u>FAQ'S</u>

## 1. DOCKER IMAGE ERROR
If you get an error during Docker image creation that says something like this:
```
WARNING: Retrying (Retry(total=4, connect=None, read=None, 
redirect=None, status=None)) after connection broken by 'SSLError
(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] 
certificate verify failed: self signed certificate (_ssl.c:992)'))
': /packages/07/51/
2c0959c5adf988c44d9e1e0d940f5b074516ecc87e96b1af25f59de9ba38/pip-23.
0.1-py3-none-any.whl
``` 

Try to change to another wi-fi network connection, since some companies avoid accesing to python pages to download certificates and dependencies.

## 2. RUNNING TESSERACT 
If you get when using tesseract that says something like this:
```
Error opening data file /home/tesseract_repos/tesseract/tessdata/eng.traineddata
Please make sure the TESSDATA_PREFIX environment variable is set to your "tessdata" directory.
Failed loading language 'eng'
Tesseract couldn't load any languages!
Could not initialize tesseract.
```

Remember to copy desired lenguage .traineddata files from _/home/tesseract_repos/tessdata_best_ to _/home/tesseract_repos/tesseract/tessdata/_

## 3. DOCKER BUILDING IMAGE HAS CACHE 
If you wish to build the image from strach, without any build cache, the following command will remove **ALL** images and containers cache in your machine.

```
docker system prune -a -f
```

