# WebTileNet
This program learns tile image as data set with chainer version pix2pix and displays tiles converted using WebDNN on Cesium.
 I created a program based on the source code published here (<https://qiita.com/knok/items/45f4bbe3f0058eba27e6>).

[日本語READMEはこちら][linkpos1]

[linkpos1]: https://github.com/makinux/WebTileNet/blob/master/README.jp.md#linkpos1 "日本語README"

## Data set creation
### Operating environment
Operation is confirmed in the following environment.

	OS: Ubuntu 14.04 64bit
	Software: python 2.7.6

### Installation method
Install each package required for execution.


	sudo pip install pillow requests

Place the source code in the .dataset directory in an arbitrary place.


### Data Set Creation Procedure
#### Setting of Map Tile Destination URL

 Create a file in which the map tile acquisition URL is set in JSON format. As a creation example, jsonSample.txt is bundled so please create it based on this.

##### Setting of teacher data and input data

	{
		"targetURL": URL from which the map tile serving as teacher data is obtained,
		"inputURL": [
			Acquisition destination URL of map tile to be input data
		]
	}

 Set the map tile acquisition destination for each teacher data and input data. For the setting of the map tile acquisition destination, refer to "Map tile acquisition destination" below.


##### Where to get the map tile

	{
		"url": base URL of map tile acquisition destination,
		"type": the type of map tile to be acquired,
		"format": format of the map tile to be acquired
	}

  "url" base URL of map tile acquisition destination

  "type" The type of map tile to be acquired. Tile map format and WMTS are "tile", WMS is "wms".

  "format" extension and tile coordinates, WMTS and WMS parameters etc settings. For details, see the example of each tile acquisition setting below.


##### tile map type map tile acquisition destination setting example (Geographical Survey Institute National Latest Picture (Seamless)):

	{
		"url": "http://cyberjapandata.gsi.go.jp/xyz/seamlessphoto/",
		"type": "tile",
		"format": "{z}/{x}/{y}.jpg"
	}

In tile map format "format", tile coordinates, extension, WMTS parameters etc. are mainly set. Please set according to the format of the acquisition destination.

Please describe the tile coordinates as follows.


	{z}: zoom level, {x}: X coordinate of tile, {y}: Y coordinate of tile


##### WMS format map tile acquisition example setting (Earthquake hazard station landslide topography map WMS service):

	{
		"url": "http://www.j-shis.bosai.go.jp/map/wms/landslide?",
		"type": "wms",
		"format": "SERVICE=WMS&VERSION=1.3.0&REQUEST=GetMap&BBOX={minx},{miny},{maxx},{maxy}&CRS=EPSG:4612&WIDTH={output_width}&HEIGHT={output_height}&LAYERS=L-V3-S300&FORMAT=image/png&TRANSPARENT=FALSE"
	}

In WMS format "format", WMS parameters are set.

Please set according to the format of the acquisition destination.

Please describe the BBOX that specifies the acquisition range in WMS parameters, WIDTH and HEIGHT that specify the size of the image to be acquired as follows.

	BBOX = {minx}, {miny}, {maxx}, {maxy}
	WIDTH = {output_width}
	HEIGHT = {output_height}

#### Create dataset
Read multiple tiles within the specified range and create learning data and test data on a tile-by-tile basis.

##### Example of execution:

	python DataSetMake_imgjoin.py 7266 7295 3206 3229 13 --outputPath datasets/test/train --inputJson ./jsonSample.txt

##### Command line arguments

	Argument1:Position in the x direction of the tile that is the starting point of the specified range
	Argument2:x-direction position of the tile which is the end point of the specified range
	Argument3:Position in the y direction of the tile that is the starting point of the specified range
	Argument4:The position in the y direction of the tile that is the end point of the specified range
	Argument5:Zoom level
	--inputJson:Specify the file in json format that sets the map tile acquisition destination URL. The default is "./jsonSample.txt".
	--outputPath:Specify the output destination directory of the data set. If there is no directory, it will be generated automatically. The default is "Data".

After execution, `{serial number}_{x coordinate}_{y coordinate}_{zoom level}.png` is generated for the number of tiles acquired in the directory specified by - outputPath.

Also, `input_image{serial number}.png, target_image{serial number}.png` is generated by connecting the acquired tiles to the directory where the program was executed.


## Learning model transformation
### Operating environment
Operation is confirmed in the following environment.

	OS: Ubuntu 14.04 64bit
	Software: python 3.6.3 (using pyenv)

### Installation method
##### 1.Install pyenv so that you can use python 3.6.3.


	cd ~
	sudo apt-get install -y git
	sudo apt-get install -y make gcc
	sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils

	git clone git: //github.com/yyuu/pyenv.git ~ / .pyenv
	git clone https://github.com/yyuu/pyenv-virtualenv.git ~ / .pyenv / plugins / pyenv-virtualenv

	vim ~ / .bashrc
	#Add the following.
	# -----from here-----
	export PYENV_ROOT = $ HOME / .pyenv
	export PATH = $ PYENV_ROOT / bin: $ PATH
	eval "$ (pyenv init -)"
	# -----So far-----

	source ~ / .bashrc

	#Check the installable version and install python 3.6.3
	pyenv install -l
	pyenv install 3.6.3
	#Check the environment of python and make sure python 3.6.3 exists
	pyenv versions


##### 2.Install WebDNN.


	cd ~
	git clone https://github.com/mil-tokyo/webdnn
	cd webdnn
	# set pyenv to use python 3.6.3 in the webdnn directory
	pyenv local 3.6.3
	python setup.py install
	sudo pip install chainer

##### 3.Install Emscripten and Eigen.


	cd ~
	# cmake If you do not need 3.4.3 or higher install it
	# I will erase the old version if there is one
	sudo apt remove cmake
	sudo apt-get install build-essential
	# Confirm acquisition destination (https://cmake.org/download/) and get it with wget
	wget https://cmake.org/files/v3.10/cmake-3.10.1.tar.gz
	tar xf cmake-3.10.1.tar.gz
	cd cmake-3.10.1
	./configure
	make
	sudo make install

	# Install Emscripten
	cd ~
	git clone https://github.com/juj/emsdk.git
	cd emsdk
	./emsdk install sdk-incoming-64bit binaryen-master-64bit
	./emsdk activate sdk-incoming-64bit binaryen-master-64bit
	source ./emsdk_env.sh

	# Install Eigen
	cd ~
	wget http://bitbucket.org/eigen/eigen/get/3.3.3.tar.bz2
	tar jxf 3.3.3.tar.bz 2
	export CPLUS_INCLUDE_PATH = $ PWD / eigen - eigen - 67 e 894 c 6 c d 8 f

##### 4.Install each package required for execution.


	cd ~ / webdnn
	sudo pip install pillow requests

##### 5.Create a workspace directory in the WebDNN directory.


	mkdir workspace

##### 6.Set the source code in the train_dump directory in the workspace directory.
Please refer to the following [WebDNN document](https://mil-tokyo.github.io/webdnn/docs/tutorial/setup.html) for environment setting.



### Learning
##### Example of execution:

	python train_joined.py --gpu 0 --dataset ./datasets/test/train --out ./models --epoch 300 --snapshot_interval 2000 --display_interval 100

##### Command line arguments

	--gpu "Number of the GPU to use. -1 learning with CPU."
	--dataset "Directory containing training data sets"
	--out "Output directory such as learning model"
	--epoch "Number of learning"
	--snapshot_interval "Output interval such as learning model"
	--display_interval "Interval of learning status console output"

Learning models and test images are output to the directory specified by --out during learning at intervals specified by --snapshot_interval.

### Model conversion
We convert the model learned with chainer into the model of WebDNN.

##### Example of execution:

	python dump_graph.py --out ./out --enc-npz models/enc_iter_10000.npz --dec-npz models/dec_iter_10000.npz

##### Command line arguments

	--out "Output directory of converted model"
	--enc-npz "Model of conversion source (designation of enc_iter_XXXX.npz name model)"
	--dec-npz "Source model (specify model with dec_iter_XXXX.npz name)"


The converted model is output to the directory specified by --out. In this program, two types of models for WebAssembly and WebGPU are output.


## Display tiles converted by Cesium
### Operating environment
Place the files in the html directory on the http server. Also, if you want to use the converted learning model, please install it in http server.

When you access the URL /index.html of the installation place, the page will be displayed.

The demo is [here](https://makinux.github.io/WebTileNet/html/index.html)

### Method of operation
#### Loading the Learning Model
Enter the URL with converted learning model in "Model Path". As the initial value, the path to the learned model that converts from the PNG elevation tile that is bundled to the CS stereoscopic figure is entered. (This model is [seamless altitude service PNG elevation tile](https://gsj-seamless.jp/labs/elev/), [CS solid figure (with 5mDEM also)](http://kouapp.main.jp/csmap/japan/csjapan.html) was used as learning data.)

Click the "model load" button to load the learning model.

#### Tile image conversion
In the "Input Tile", enter the destination URL of the tile

##### Input example (seamless altitude service PNG elevation tile):

	https://gsj-seamless.jp/labs/elev/m/{z}/{y}/{x}.png

##### Please describe the tile coordinates as follows.

	{z}: zoom level, {x}: X coordinate of tile, {y}: Y coordinate of tile

Specify the range of the map tile to be converted. Tile coordinates in the vicinity of Chiba prefecture are input as an initial value. For Tile range X, enter the X coordinate of the tile, the Y coordinate of the tile in the Tile range Y, and the zoom level of the tile in the Tile Z.

 Click convert to start converting tiles.

 When you click clear, converted tiles on Cesium will be deleted.
