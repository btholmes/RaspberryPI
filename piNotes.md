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


## Set up RaspberryPI WebCam server (Regular webcam, and then PiCamera extra steps at bottom)  
    sudo apt-get update  
    sudo apt-get upgrade  

    **Remove packages that can conflict with newer versions**  
    sudo apt-get remove libavcodec-extra-56 libavformat56 libavresample2 libavutil54  

    wget https://github.com/ccrisan/motioneye/wiki/precompiled/ffmpeg_3.1.1-1_armhf.deb  
    sudo dpkg -i ffmpeg_3.1.1-1_armhf.deb  

    sudo apt-get install curl libssl-dev libcurl4-openssl-dev libjpeg-dev libx264-142 libavcodec56 libavformat56 libmysqlclient18 libswscale3 libpq5  

    wget https://github.com/Motion-Project/motion/releases/download/release-4.0.1/pi_jessie_motion_4.0.1-1_armhf.deb  
    sudo dpkg -i pi_jessie_motion_4.0.1-1_armhf.deb  

    **Makes changes to motion.conf file**  
    sudo nano /etc/motion/motion.conf  

    Find the following lines and change them to the following.  
    daemon on  
    stream_localhost off  
    Note: Change the following two lines from on to off if you’re having issues with the stream freezing whenever motion occurs.  
    output_pictures off  
    ffmpeg_output_movies off  
    Optional (Don’t include the text in brackets)  
    stream_maxrate 100 (This will allow for real-time streaming but requires more bandwidth & resources)  
    framerate 100 (This will allow for 100 frames to be captured per second allowing for smoother video)  
    width 640 (This changes the width of the image displayed)  
    height 480 (This changes the height of the image displayed)  

    **Now edit the motion file**  
    sudo nano /etc/default/motion  

    Find the following line and change it to the following:  
    start_motion_daemon=yes  
    Once you’re done simply save and exit by pressing ctrl+x then y.  

    With the camera connected run :  
        sudo service motion start
        sudo service motion stop (to stop service)  
        **Now can see the webstream at IP address of your PI** 
            192.168.1.103:8081
        *If web page isn't loading try*  
            sudo service motion restart  


 * Extra steps for the PiCamera  
    Make sure camera is enables  
        sudo raspi-config  
    Open modules file  
        sudo nano /etc/modules  
    Enter this at bottom of file if not there  
        bcm2835-v4l2  (save and exit with ctrl x then y)  
    sudo reboot

    Now check it out at 192.168.1.103:8081  

## Enable external Access to PiCamera stream 

    In order to enable external access to the Raspberry Pi Webcam Server we will need to change some settings in the router. However, all routers are designed differently so you may need to look up instructions for your brand of router. Please note, opening ports to the internet comes with a security risk.  
    If you need a more indepth guide then be sure to take a look at my guide on how to setup Raspberry Pi port forwarding and dynamic DNS.  
    This is what I did on mine in order to get it to work. My router is an AC1750 TP-Link Router.  
    Go to the Router admin page (This will typically be 192.168.1.1 or 192.168.254)  
    Enter the username and password. Default typically is admin & admin.  
    Once in go to forwarding->Virtual Server and then click on add new  
    In here enter:  
    **Service port:** In this case 48461  
    **IP Address:** 192.168.1.103 (Address of your Pi)  
    **Internal Port:** We want this to be the same as the webcam server so make it 8081  
    **Protocol:** All  
    **Status:** Enabled  
    These settings will route all traffic destined for port 48461 to the webcam server located at the IP address and port you provided. (192.168.1.103:8081)  
    You should now be able to connect to the Raspberry Pi webcam stream outside your network. You may need to restart the router for changes to take effect.  
    Router Port Fowarding  
    If you’re unable to connect outside your local network then you can try the following:  
    Check your router settings and confirm they are correct.  
    Check your IP hasn’t changed (Some IPs will provide you with a dynamic IP rather than a static IP) You can  setup something called dynamic dns to counter this you can find out more information via the link mentioned above.  
    Restart the router.  




# Note
    cmd < goes to settings in chrome 
