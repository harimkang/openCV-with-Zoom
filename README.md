# opencv-with-zoom
![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fharimkang/openCV-with-Zoom) ![License](https://img.shields.io/github/license/harimkang/opencv-with-zoom?style=plastic) ![Stars](https://img.shields.io/github/stars/harimkang/opencv-with-zoom?style=social)


![love and jo](https://user-images.githubusercontent.com/38045080/88261899-1d189380-cd02-11ea-9058-740b2f6a48b4.png)

## Blog for Korean

The READ ME contents in Korean can be found at the address below.

<https://davinci-ai.tistory.com/8>

## Environment

- Python3
- opencv-python

## Installation

Install the dependency.
```sh
$ pip install opencv-python
```

Clone Repository...

```sh
$ mkdir project
$ cd project
$ git clone https://github.com/harimkang/opencv-with-zoom.git
$ cd opencv-with-zoom
```

## Start

```sh
$ python Camera.py
```
And you can zoom through the double click.

You can exit through the q button.


## Goal

The repository will bring the camera's screen through openCV to show it, add zoom-in and zoom-out functions, and write code to use the zoom function using screen touch events. These codes are written in python code.

![opencv%20with%20zoom/Untitled.png](opencv%20with%20zoom/Untitled.png)

The part related to the function of the repository in the Activity Diagram is as shown above.

## Create openCV streaming screen

First of all, write code to show the live streaming screen using openCV.
```python
import cv2
class Camera:
    def __init__(self, mirror=False):
        self.data = None
        self.cam = cv2.VideoCapture(0)

        self.WIDTH = 640
        self.HEIGHT = 480

        self.center_x = self.WIDTH / 2
        self.center_y = self.HEIGHT / 2
        self.touched_zoom = False

        self.scale = 1
        self.__setup()

        self.recording = False

        self.mirror = mirror

    def __setup(self):
        self.cam.set(cv2.CAP_PROP_FRAME_WIDTH, self.WIDTH)
        self.cam.set(cv2.CAP_PROP_FRAME_HEIGHT, self.HEIGHT)
        time.sleep(2)

    def stream(self):
        # streaming thread function
        def streaming():
            # The actual threaded function
            self.ret = True
            while self.ret:
                self.ret, np_image = self.cam.read()
                if np_image is None:
                    continue
                if self.mirror:
                    # Inverted left and right in mirror mode
                    np_image = cv2.flip(np_image, 1)
                self.data = np_image
                k = cv2.waitKey(1)
                if k == ord('q'):
                    self.release()
                    break

        Thread(target=streaming).start()

    def show(self):
        while True:
            frame = self.data
            if frame is not None:
                cv2.imshow('Davinci AI', frame)
            key = cv2.waitKey(1)
            if key == ord('q'):
                # q : close
                self.release()
                cv2.destroyAllWindows()
                break

    def release(self):
        self.cam.release()
        cv2.destroyAllWindows()


if __name__ == '__main__':
    cam = Camera(mirror=True)
    cam.stream()
    cam.show()
```

I wrote Camera class that sends streaming screen of personal camera through methods provided in openCV. When you run the code, it simply displays the video from the camera connected to your computer.

I want to add basic zooming, zooming through touch events.

## Adding basic zoom features

First of all, before we start writing the code, let's easily represent it with a picture. Since we are going to implement zoom function through touch event, we wrote this in consideration.

The circle means the touched position, and it looks at the size of the top and bottom and sides based on the position and considers the small part as half of the new frame to find and cut the frame of the new size.

![opencv%20with%20zoom/Untitled%201.png](opencv%20with%20zoom/Untitled%201.png)

Now let's write the code that handles the basic zoom functions. I know that openCV doesn't provide a zoom-related method, so I wrote a code that cuts and resizes the screen to fit a variable that is a multiple of scale.
```python
    def __zoom(self, img, center=None):
            # actual function to zoom
            height, width = img.shape[:2]
            if center is None:
                #   Calculation when the center value is the initial value
                center_x = int(width / 2)
                center_y = int(height / 2)
                radius_x, radius_y = int(width / 2), int(height / 2)
            else:
                #   Calculation at a specific location
                center_x, center_y = center
    
                center_x, center_y = int(center_x), int(center_y)
                left_x, right_x = center_x, int(width - center_x)
                up_y, down_y = int(height - center_y), center_y
                radius_x = min(left_x, right_x)
                radius_y = min(up_y, down_y)
    
            # Actual zoom code
            radius_x, radius_y = int(self.scale * radius_x), int(self.scale * radius_y)
    
            # size calculation
            min_x, max_x = center_x - radius_x, center_x + radius_x
            min_y, max_y = center_y - radius_y, center_y + radius_y
    
            # Crop image to size
            cropped = img[min_y:max_y, min_x:max_x]
            # Return to original size
            new_cropped = cv2.resize(cropped, (width, height))
    
            return new_cropped
```
The code is divided into two types when the center touched and the zoom function is used. It takes the size of the image, finds the center value, calculates the size according to the scale, crops it accordingly, and increases the size to the original size to return.

```python
def touch_init(self):
    self.center_x = self.WIDTH / 2
    self.center_y = self.HEIGHT / 2
    self.touched_zoom = False
    self.scale = 1
        
def zoom_out(self):
    # scale 값을 조정하여 zoom-out
    if self.scale < 1:
        self.scale += 0.1
    if self.scale == 1:
        self.center_x = self.WIDTH
        self.center_y = self.HEIGHT
        self.touched_zoom = False
    
def zoom_in(self):
    # scale 값을 조정하여 zoom-in
    if self.scale > 0.2:
        self.scale -= 0.1

def zoom(self, num):
    if num == 0:
        self.zoom_in()
    elif num == 1:
        self.zoom_out()
    elif num == 2:
        self.touch_init()
```

The zoom in and out functions are structured to function by adjusting the scale value.
```python
if self.touched_zoom:
    np_image = self.__zoom(np_image, (self.center_x, self.center_y))
else:
    if not self.scale == 1:
        np_image = self.__zoom(np_image)
```
Put the above code before the middle of the stream method (self.data = np_image) to check whether the zoom by touch function is already executed or the basic zoom function is executed.

## Adding zoom function by Touch-Event
```python
def show(self):
    while True:
        frame = self.data
        if frame is not None:
            cv2.imshow('Davinci AI', frame)
            cv2.setMouseCallback('Davinci AI', self.mouse_callback)
        key = cv2.waitKey(1)
        if key == ord('q'):
            # q : close
            self.release()
            cv2.destroyAllWindows()
            break
def mouse_callback(self, event, x, y, flag, param):
    if event == cv2.EVENT_LBUTTONDBLCLK:
        self.get_location(x, y)
        self.zoom_in()
    elif event == cv2.EVENT_RBUTTONDOWN:
        self.zoom_out()
```

In the code above, I added the red background cv2.setMouseCallback ('Davinci AI', self.mouse_callback) to the show function. That code means you assign a function called self.mouse_callback to a function that handles touch events on the screen.

The following mouse_callback function is a function that handles touch events. The code is configured to be zoom_in when double-clicking the left mouse and zoom_out when double-clicking the right mouse.

![opencv%20with%20zoom/Untitled%202.png](opencv%20with%20zoom/Untitled%202.png)

### Problem occurred!

Touch zoom at the edges tends to distort the screen or to zoom in too suddenly, so we want to fix it so that it's proportional to touch zoom.

As shown in the figure below, during touch event of the edge part, the touch position is adjusted according to a certain ratio to prevent distortion. Enable magnification while maintaining a reasonable ratio.

![opencv%20with%20zoom/Untitled%203.png](opencv%20with%20zoom/Untitled%203.png)

The code above is shown below.
```python
def __zoom(self, img, center=None):
    # actual function to zoom
    height, width = img.shape[:2]
    if center is None:
        #   Calculation when the center value is the initial value
        center_x = int(width / 2)
        center_y = int(height / 2)
        radius_x, radius_y = int(width / 2), int(height / 2)
    else:
        #   Calculation at a specific location
        rate = height / width
        center_x, center_y = center

    #   Calculate centroids for ratio range
    if center_x < width * (1-rate):
        center_x = width * (1-rate)
    elif center_x > width * rate:
        center_x = width * rate
    if center_y < height * (1-rate):
        center_y = height * (1-rate)
    elif center_y > height * rate:
        center_y = height * rate

    center_x, center_y = int(center_x), int(center_y)
    left_x, right_x = center_x, int(width - center_x)
    up_y, down_y = int(height - center_y), center_y
    radius_x = min(left_x, right_x)
    radius_y = min(up_y, down_y)

    # Actual zoom code
    radius_x, radius_y = int(self.scale * radius_x), int(self.scale * radius_y)

    # size calculation
    min_x, max_x = center_x - radius_x, center_x + radius_x
    min_y, max_y = center_y - radius_y, center_y + radius_y

    # Crop image to size
    cropped = img[min_y:max_y, min_x:max_x]
    # Return to original size
    new_cropped = cv2.resize(cropped, (width, height))

    return new_cropped
```
Added a red background code to the __zoom function responsible for the zoom function.

![opencv%20with%20zoom/Untitled%204.png](opencv%20with%20zoom/Untitled%204.png)

## Additional features


In addition, we want to add a function to capture and save the current image and to record a video.
```python
def save_picture(self):
    # Function to save image
    ret, img = self.cam.read()
    if ret:
        now = datetime.datetime.now()
        date = now.strftime('%Y%m%d')
        hour = now.strftime('%H%M%S')
        user_id = '00001'
        filename = './images/cvui_{}_{}_{}.png'.format(date, hour, user_id)
        cv2.imwrite(filename, img)
        self.image_queue.put_nowait(filename)
```
First is the image capture function. You need to create a folder called images first.
```python
def record_video(self):
    # Video recording function
    fc = 20.0
    record_start_time = time.time()
    now = datetime.datetime.now()
    date = now.strftime('%Y%m%d')
    t = now.strftime('%H')
    num = 1
    filename = 'videos/cvui_{}_{}_{}.avi'.format(date, t, num)
    while os.path.exists(filename):
        num += 1
        filename = 'videos/cvui_{}_{}_{}.avi'.format(date, t, num)
    codec = cv2.VideoWriter_fourcc('D', 'I', 'V', 'X')
    out = cv2.VideoWriter(filename, codec, fc, (int(self.cam.get(3)), int(self.cam.get(4))))
    while self.recording:
        if time.time() - record_start_time >= 600:
            self.record_video()
            break
        ret, frame = self.cam.read()
                    if ret:
            if len(os.listdir('./videos')) >= 100:
                name = self.video_queue.get()
                if os.path.exists(name):
                    os.remove(name)
            out.write(frame)
            self.video_queue.put_nowait(filename)
        k = cv2.waitKey(1)
        if k == ord('q'):
            break
```
Next is the video recording function. The code is configured to be saved as another file when recording for more than 10 minutes. It also queues to delete over 100 files. (Method to adjust file size not too big)
