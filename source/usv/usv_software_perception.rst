Perception
==========

Start using OpenCV with Python
--------------------------------------------------

Authors:
 
* Alejandro Hernández
* Eduardo 
* Alberto
* Pequeño
* Alexa Arreola

First practice: Canny
------------

Canny() is used to detect edges in an image.

.. code-block:: python

  import numpy as np
  import cv2 #importamos opencv2
  import sys


  #imread loads an image from a path, supports .jpg, .jpeg, .png, .tiff, .exr, etc... : https://docs.opencv.org/3.4/d4/da8/group__imgcodecs.html#ga288b8b3da0892bd651fce07b3bbd3a56
  image = cv2.imread('C:\\Users\\oscar\\Downloads\\img1.jpeg',0) # 0 is used to read in grayscale mode
  #Comment: the path's format can change
  #Ex.: image = cv2.imread('C:/Users/oscar/Downloads/img1.jpeg', 0) 


  #canny = cv2.Canny(image, 20, 170)
  canny = cv2.Canny(image, 50, 120) #v2.Canny(image, T_lower, T_upper)..T_lower/T_upper: threshold value in Hysteresis Thresholding
  #https://www.geeksforgeeks.org/python-opencv-canny-function/#:~:text=Canny()%20Function%20in%20OpenCV,the%20edges%20in%20an%20image.

  #imshow is used to display an image, it fits into image's size 
  cv2.imshow('Canny', canny) # cv2.imshow(window_name, image)

  #displays window for x miliseconds or until key is pressed
  cv2.waitKey(0) #close window until any key is pressed
  #Ex. for 5 seconds: cv2.waitKey(5000)

  cv2.destroyAllWindows() #Destroy all windows created
  
