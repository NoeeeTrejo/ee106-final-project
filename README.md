# ee106-final-project

## Goal: The end goal of this project was to showcase how robots can interact with humans, and play games. 

## Why 
Proof of concept! Development in this area could allow possibly empower people with movement disabilities and give them a way to independently play games with others. Furthermore, this could allow for virtual gaming tournaments with real pieces, interactive dungeons and dragons sessions with real parts as well. 


This is an interesting project because it transcends the boundaries of the physical world and the digital world. By getting a robot to play a game in the real world, it showcases how we can use robots to create a form of entertainment (a dice game in this case). The problem solved was making the robot see with it's camera and use that information to pick up the dice. 

## How: 

### How do we get the robot to know where the dice are?
We set up a configuration that is ready to pick up a dice, so the sawyer bot is ready and sets its camera up to view where the dice is. Once it’s all set up, the bot uses opencv’s contouring to figure out where the dice is. Upon doing that, we translate the picture’s coordinates and shift the robot left, or right according to where the dice is so that the dice ends up in the center of the camera view. Once the dice is at the center, we initiate the pick up sequence.


### In what real-world robotics applications could the work from your project be useful?
The way we approached the problem was very unique and that that approach can be used in other applications. Instead of translating to cartesian coordinates using relativity makes the problem a lot more simple, so we stuck to the robot’s frame, rather than the world frame since it made the problem a lot more simple. 


## Design: 
### What design criteria must your project meet? What is the desired functionality? 
- **Computer Vision**: to read, recognize, and locate the dice
- **Consistency:** since we plan on making a game, we want to be able to have consistent success so we made checkpoints that happen along the problem to ensure we're on the right path. 

### What design choices did you make when you formulated your design? What trade-offs did you have to make?

**Path Finding**: We created our own path finding solution, by moving closer and closer to the dice to create a checkpoint where we can then pick up the dice. 
Tradeoffs: 
    - By making our own path finding algorithm we got to learn the knitty gritty of what it takes to move a robot with vision and how difficult tasks involving small objects are 
    - By having a standardized way to pick things up, we were able to have quicker functions that gave us quick paths rather than suboptimal paths using other pathfinding algorithms that would make us take ridiculous paths to pick up the dice
    
**Robot(s)**: We at fist were ambitious and wanted to use multiple robots, but we found it easier to just use a single robot to simulate a full game. Though it adds to the entertainment aspect, but could be a possible project for the future.

## Implementation

We used a sawyer with the default gripper. For rolling the dice we 3D printed a container. 	![cup we 3d printed](/imgs/cup.jpeg)
We also used a plate that had curved edges around the sides. The dice also rolled less far off when it landed on the plate. ![plate](/imgs/10.jpg)


### Describe any software you wrote in detail. Illustrate with diagrams, flow charts, and/or other appropriate visuals. This includes launch files, URDFs, etc. 
**Reading the dice**:
- We started with using concepts of Lab 6 to read dice numbers on still images
- OpenCV functions to find the dice and count circular blobs
- Outputs a number to add to total count
- Started to use camera to capture images to be processed

#### How it works: 
0. A picture is taken from the video stream

![Normal black and white picture of dice](/imgs/1.jpeg)
	
1. We first apply a background remover using a threshold number. Then blur it to remove noise.

![Gaussian Blurred picture of dice to reduce noise](/imgs/2.jpeg)

2. Then we use opencv to identify the clusters

![Opencv's contour to find the dice](/imgs/3.jpeg)
	
3. From the clusters we apply a box identifier, with a threshold of how large it can be to eliminate the frame rectangle.

![Creating boxes around the dice](/imgs/4.jpeg)
	
4. After individual dice are identified a snapshot of each dice is taken. Then zoomed in to have more accurate counts of the dice

|:------------------------------:|:--------------------------------:|: ----------------------------:|:-------------------------------:|:-----------------:|
![Dice counting](/imgs/5.jpeg)  |  ![Dice counting2](/imgs/6.jpg)  | ![Dice counting3](/imgs/7.jpg) | ![Dice counting4](/imgs/8.jpg) | ![Dice counting5](/imgs/9.jpg) |

![Dice counting](/imgs/5.jpeg) 

![Dice counting2](/imgs/6.jpg) 

![Dice counting3](/imgs/7.jpg) 

![Dice counting4](/imgs/8.jpg) 

![Dice counting5](/imgs/9.jpg)

### Picking up the dice:

The high level overview of our project works like the following. From a fixed starting position, a container with a dice is picked up. The rolling motion is a preset set of instructions. Then the robot moves so that the rolled dice is in frame. It takes a few seconds to read the dice and locate the dice. Then it determines its location and picks up the dice

To pick up the dice, we use a controller. 

### Camera Position Controller
![Camera Position Controller Diagram](/imgs/diagram.png)

Sudo code for how this would work: 
```
# Get an intial read
image_center = get_img()
dice_pos = get_dice_pos()
error = dice_pos - image_center # In (x, y)

# move the camera, then get another read and update the error
while error not within a threshold: 
	move_camera(error)
	dice_pos = get_dice_pos()
	image_center = get_img()
	error = dice_pos - image_center # In (x, y)

```
![controller demo](/imgs/adjustment_demo.gif) 

- We got the bot to get the dice into the center of the frame, and record it’s x and y position
- Look at the error of where the dice is and the center of the image, and that tells us how much to adjust in order to center the dice in the middle
- We do this via x and y positions relative to the camera, and we maintain a constant height
- Once we got that, we use that position to tell it where to grab with a constant offset from the camera position

## Results

### How did it end up? 
In the end we completed our project so that it could play a dice game with a singular dice. It is successful with rolling the dice and identifying the dice numbers

![Working Project! ](/imgs/demo.gif)
![Working Project2! ](/imgs/demo2.gif)

## Conclusion 
Our results met our design criteria very well! We were able to play with a single dice, but so the dice game needed to be simplified. A future project could be working on a version that simply looks at the dice and picks them up, without using checkpoints. Also, deciding on which dice to pick up first could also be a possible factor as well. Regardless, our solution met our design criteria of being able to use computer vision to allow the robot to identify, locate, and pick up a dice and place in a container. 


### Manipulation:
- Dice can be rolled too far from the target area or even off the table
- Orientation of the dice needed to be taken into account
- Final gripper position depends on previous position; hysteresis
 

### Reading the dice:
- Incomplete JPEG: camera takes snapshots and corrupt JPEG makes dice reading difficult
- General inconsistency with coordinates
- Adjusting blurring to deal with noise as well as sensitivity of circular shapes to correctly count dots
- Multiple dice mess up the camera (which die to center on)

### "Hacks"
Our implementation uses checkpoints which are configurations for the sawyer bot that tell us we're on the right path. They're used in our case to add another layer of consistency for the robot, making it easier to replicate errors. For example, we use a sawyer tuck, to essentially calibrate the bot for picking the dice up, and then we continue off that and have different checkpoints: 
- cup pick up (to ensure consistent grabbing for the cup)
- cup drop off (to ensure that the dice falls directly down)
- dice pick up (to ensure dice is picked up according to it's orientation)
- dice drop (using random height) 

### Improvements
1. Better Sensing: 
	- Pick up the dice using only a single view, without centering the dice 
	- Using a method that could give us cartesian coordinates relative to the robot
	- Detect the cup similar to the dice, because it has a fixed position
2. Multiple Robot Integration: 
	- Getting multiple robots to play against eachother 
	- This would also build off better sensing
	> Side Note: This doesn't seem too bad! Using ROS it could be very easy to share flags of who's turn it is, and allow the robots to use that, and incorperate object detection to make sure it doesn't hit anything. 

3. Multiple Dice 
	- Something else that we ran out of time and weren't able to do is making the robot pick up multiple dice, luckily this implementation would be very simple and would just ask the robot to repeat the task until there's no more die on the camera. 

4. Custom Game Integration
	- With some manipulation and standardized pick up actions, a more complex game could be created! This could possibly allow for virtual gaming tournaments with real pieces moving, an interactive dungeons and dragons session, and possibly could empower people with movement disabilities and give them a way to independently play games with others. 


## Team
- Adan Lopez Calderon: 4th year studying EECS
<br> </br>
![adan](/imgs/adan.jpeg)

- Noe Trejo-Cruz: 4th year studying EECS 
<br> </br>
![noe](/imgs/noe.png)

- Khanh Pham: 4th year studying Physics
<br> </br>
![khanh](/imgs/khanh.jpeg)

- Angela Gao: 3rd year studying EECS and business. 
<br> </br>
![angela](/imgs/angela.jpeg)




### Contributions: 
- Adan Lopez Calderon: Adan worked on coding the vision part of the project to count the number of dots, he also helped with writing the starting openCV functions with Angela, wrote script for the error adjustment
- Noe Trejo-Cruz: Worked with Khanh to create the checkpoints, set up a script to run everything, created launch files, mapped out the planning for the project, created scripts for specific instructions, created slides, and wrote the github readme.
- Khanh Pham: coded the vision part to detect the dice and count the number of dots. He also helped with writing the controller and testing the Sawyer arm with pickup up the dice. 
- Angela Gao: started the vision part with basic openCV functions that were later modified to be the final project. Also helped with coordinating work times and finishing up writing the website.
 

