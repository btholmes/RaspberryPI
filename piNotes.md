# Notes on Raspberry Pi Use
    http://www.pyimagesearch.com/2016/04/18/install-guide-raspberry-pi-3-raspbian-jessie-opencv-3/
 * Enable SSH 
    Enter sudo raspi-config in the terminal, 
    first select Interfacing options, 
    then navigate to ssh, 
    press Enter and select Enable or disable ssh server.

 * Set up PyCamera
  * Make sure there is internet connectivity
    sudo apt-get update 
    sudo apt-get upgrade 
    sudo raspi-config 
        select enable camera on interface 
    raspistill -o camera.jpg
    
 * Set up Python and OpenCV on raspberryPi 3 
    sudo raspi-config 
        select Expand filesystem 
    sudo reboot 
    df -h (Shows filesystem expanded to include space on microsd card)
    sudo apt-get purge wolfram-engine (delete wolfram-engine to free space for opencv)
    
    sudo apt-get install build-essential cmake pkg-config
    sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
    sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
    sudo apt-get install libxvidcore-dev libx264-dev
    sudo apt-get install libgtk2.0-dev
    sudo apt-get install libatlas-base-dev gfortran
    sudo apt-get install python2.7-dev python3-dev
    
  * Now download open CV source code 
    cd ~
    wget -O opencv.zip https://github.com/Itseez/opencv/archive/3.1.0.zip
    unzip opencv.zip
    wget -O opencv_contrib.zip https://github.com/Itseez/opencv_contrib/archive/3.1.0.zip
    unzip opencv_contrib.zip
    
  * For Python 
    wget https://bootstrap.pypa.io/get-pip.py
    sudo python get-pip.py

  * Virtualenv and env wrapper 
    sudo pip install virtualenv virtualenvwrapper
    sudo rm -rf ~/.cache/pip
    export WORKON_HOME=$HOME/.virtualenvs
    source /usr/local/bin/virtualenvwrapper.sh

    echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.profile
    echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.profile
    echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.profile

    source ~/.profile

    mkvirtualenv cv -p python2
    mkvirtualenv cv -p python3

    **NOW TEST ENV** 
    source ~/.profile
    workon cv

  * Install Numpy 
    pip install numpy
    
  * Compile and Install OpenCV
    workon cv
    cd ~/opencv-3.1.0/
    mkdir build
    cd build

cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-3.1.0/modules \
-D BUILD_EXAMPLES=ON ..

    Scroll down to Python 2 and 3 sections, make sure valid paths are used for Interpreter, Libraries, Numpy, 
    and packages path (Interpreter and Numpy are in virtualEnv path) 

    make -j4 (compile openCV with 4 cores)
    make clean 
    make 

  * Almost done now 
    sudo make install
    sudo ldconfig


  **Python 2.7**
    Open CV should be installed at /usr/local/lib/python2.7/site-pacakges
    **Verify with**
    ls -l /usr/local/lib/python2.7/site-packages/
    total 1852
    -rw-r--r-- 1 root staff 1895772 Mar 20 20:00 cv2.so  (Notice the cv2.so bindings) 

    If it's not there, check /usr/local/lib/python2.7/dist-packages

    **Now sim-link openCV bindings in virtual env**
    cd ~/.virtualenvs/cv/lib/python2.7/site-packages/
    ln -s /usr/local/lib/python2.7/site-packages/cv2.so cv2.so

  **Python 3**
    ls -l /usr/local/lib/python3.4/site-packages/
    total 1852
    -rw-r--r-- 1 root staff 1895932 Mar 20 21:51 cv2.cpython-34m.so

    For some reason when compiling openCV bindings for python 3, file is named cv2.cpython-34m.so
    Good idea to rename this using 

    cd /usr/local/lib/python3.4/site-packages/
    sudo mv cv2.cpython-34m.so cv2.so

    **Now symlink openCV bindings for virtual env**
    cd ~/.virtualenvs/cv/lib/python3.4/site-packages/
    ln -s /usr/local/lib/python3.4/site-packages/cv2.so cv2.so

 * Test OPENCV
    source ~/.profile 
    workon cv
    python
    >>> import cv2
    >>> cv2.__version__
    '3.1.0'
    >>>
    
 * To free up space and delete openCV directories 
    rm -rf opencv-3.1.0 opencv_contrib-3.1.0


# Accessing Data From Raspberry Pi Camera 
    http://www.pyimagesearch.com/2015/03/30/accessing-the-raspberry-pi-camera-with-opencv-and-python/

 * Install Picamera 
    source ~/.profile
    workon cv
    pip install "picamera[array]" (need the optional array submodule to utilize openCV) 

    touch test_image.py 
        -paste this code 
        # import the necessary packages
        from picamera.array import PiRGBArray
        from picamera import PiCamera
        import time
        import cv2

        # initialize the camera and grab a reference to the raw camera capture
        camera = PiCamera()
        rawCapture = PiRGBArray(camera)

        # allow the camera to warmup
        time.sleep(0.1)

        # grab an image from the camera
        camera.capture(rawCapture, format="bgr")
        image = rawCapture.array

        # display the image on screen and wait for a keypress
        cv2.imshow("Image", image)
        cv2.waitKey(0)

    **Test file**
    python test_image.py


  * Test video stream 
        touch test_video.py 

        # import the necessary packages
        from picamera.array import PiRGBArray
        from picamera import PiCamera
        import time
        import cv2

        # initialize the camera and grab a reference to the raw camera capture
        camera = PiCamera()
        camera.resolution = (640, 480)
        camera.framerate = 32
        rawCapture = PiRGBArray(camera, size=(640, 480))

        # allow the camera to warmup
        time.sleep(0.1)

        # capture frames from the camera
        for frame in camera.capture_continuous(rawCapture, format="bgr", use_video_port=True):
        # grab the raw NumPy array representing the image, then initialize the timestamp
        # and occupied/unoccupied text
        image = frame.array
    
        #  show the frame
        cv2.imshow("Frame", image)
        key = cv2.waitKey(1) & 0xFF

        # clear the stream in preparation for the next frame
        rawCapture.truncate(0)

        # if the `q` key was pressed, break from the loop
        if key == ord("q"):
        break

    **Test video file**
    python test_video.py 


## Set up RaspberryPI WebCam server 
    



# Note
    cmd < goes to settings in chrome 
