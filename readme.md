

<div align="center">
<h1>Building an Isometric Game in C</h1> 
By Sheldon Chong | 12 March 2025
</div>


<div align="center">

![image](https://github.com/user-attachments/assets/2f77d3c7-c6b4-4ea7-841d-5d14df89d319)

</div>



## Introduction: What is so_long?

so_long is one of game-development projects in 42’s Core Program. The objective of so_long is to create a 2D, interactive top-down game, using a lightweight C graphics library called MiniLibX.

- Players must use keys to move their player around and collect collectibles, to make their way towards the exit. 
- As a bonus, players may also incorporate various elements such as animation, enemies and enemy AI, amongst other things.

The conventional approach to this project is to make a flat, 2D grid-based game, as I’ve observed from many cadets who worked on the same project.

## My approach to so_long

https://github.com/user-attachments/assets/27940a89-dbe1-47da-ba11-e2a3add560e2

Having prior experience with game development, I had the ambitious idea of attempting to make my project a real-time, side-scrolling, stealth-based, isometrically-rendered game for my project. 

The game features the following:

- **Enemy AI**, designed in a way that would allow the player to remain undetected under certain conditions. 
	- By default, enemies are idle and look around randomly
 	- When the player falls within their line of sight, they become agitated, and chase after the player.
  	- If the player disappears from their line of sight by hiding behind a wall, they return back to normal state.
- **A camera** that smoothly follows the player alongside, and is offsetted by the position of the mouse.
- **Isometric appearance:** I designed assets in a form which connects seamlessly along with other tiles on a diagonally angled axis.
	- All calculations done in the engine have to be mapped to diagonal space, before being rendered to the screen.

This article serves as a walkthrough that will explain how I’ve gone about implementing these features.

---

## Basic term definitions and distinctions for 2D games (online examples)

<table border="1">
    <thead>
        <tr>
            <th>Side-Scroller</th>
            <th>Fixed-screen</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <img src="images/e8fd4456_parallax-scrolling-sonic.gif" alt="Parallax scrolling Sonic" width="450">
            </td>
            <td>
                <img src="images/97a2d46c_Pacman.gif" alt="Pacman" width="500">
            </td>
        </tr>
        <tr>
            <td>Camera follows player, as they traverse through the world.</td>
            <td>Camera stays in place, or the entire world is viewable without requiring the camera to track the player.</td>
        </tr>
    </tbody>
</table>


<br/>

<table border="1">
    <thead>
        <tr>
            <th>Isometric rendering</th>
            <th>Top-down/side-view rendering</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <img src="images/120178a8_image.png">
                <img src="images/4e1bbc70_image.png">
            </td>
            <td>
                <img src="images/301f722b_image.png">
                <img src="images/f691dab9_image.png">
            </td>
        </tr>
        <tr>
            <td>Tiles are rendered diagonally (or diamond-shaped). Gives the illusion of depth and dimension.</td>
            <td>Tiles can typically fit within a square, and are typically drawn from top-to-bottom, left-to-right.</td>
        </tr>
    </tbody>
</table>

<br/>

---

# 📦 Displaying a grid isometrically

![Untitled](images/1cfbfcbf_Untitled.png)

<br/>

There are several methods of rendering pixels isometrically. I will cover what I consider to be the simplest approach. The general concept to map a pixel to an isometric plane is to simply give a ratio/relationship between the X and Y coordinates. 

- a certain amount of X is equal to a certain amount of Y and vice-versa 

- By increasing the X, the Y should also change by a percentage of the X 

This concept is illustrated below

| This is the X and Y axis of a flat 2D plane. In the middle is the player | ![Untitled](images/c625b4e7_Untitled.png) |
|--------------|--------------------|
| If the player were to increase their X value, they would move towards the right | ![Untitled](images/7004aaf0_Untitled.png) |
| Now, this is the Isometric grid that you may be familiar with | ![Untitled](images/e23b0528_Untitled.png) |
| Notice how it has a skewed version of the X and Y axis | ![Untitled](images/86419ad8_Untitled.png) |
| This is the player at the center of the grid: | ![Untitled](images/4bb7aae7_Untitled.png) |
| By increasing the player’s X, the player will move forward on the X axis | ![Untitled](images/62fb7618_Untitled.png) |
| If we observe the player’s movement on the canvas, noticed how by increasing the X, the player moves somewhat diagonally. This contrasts how we saw the player move in a 2D grid when only its X position increased. In that instance, the player only move to the right | ![Untitled](images/c9703a07_Untitled.png) |
| We can observe that by increasing the X value of the player, the Y increases by a certain amount as well. This means that we need to develop a formula correlates a player’s X with its Y. | ![Untitled](images/ea1a10ac_Untitled.png) |

Now, we need to find a way to put this theory into action. In a game, we have what I shall term “true coordinates”, which are is the variable that keeps track of the position an object, such as the player.

- For example, (0,0) means the player is at the center.

However, as established in an isometric display, the position that the player is mapped onto the screen will be different than the true value

This implies that there are two sets of coordinates we need to care about: The true coordinates (where the object actually is), and the isometric coordinates (where the object is rendered on the screen) *


<br/>

## 💡 **Basic isometric mapping formula**

This is the formula to get a X position mapped onto a Isometric plane, given the X and Y coordinates of the player: 

$m$ represents the “slope”: how much a increase in $X$ results in an increase $Y$. 

$$x\ =\ m\left(x_{1}+y_{1}\right)$$

And the formula to get the Y position of the player

$$ y = \frac{\left(x_{1}-y_{1}\right)}{m} $$

Combine both:

$$(\{x, y\}) = (\{m(x_1 + y_1)), ((x_1 - y_1) / m\})$$

With this formula, we can create the following function that will give us a new set of coordinates in an isometric plane based on a given set of coordinates.

(No struct):

```c
void iso_map(int x, int y)
{
	int iso_x = (x + y) * 1.4; 
	int iso_y = (x - y) / 1.4;
	
	// do something
}
```


(Using a struct to group coordinates):

```c
t_xy iso_map(t_xy pos)
{
	t_xy iso_pos;
	iso_pos.x = (pos.x + pos.y) * 1.4;
	iso_pos.y = (pos.x - pos.y) / 1.4;
	return iso_pos;
}
```

<br/>

Using [Desmos](images/https://www.desmos.com/calculator/gewxhbpklh), we can simulate the relationship live using this same formula.
$x_1$ and $y_1$ are the true position, while the green dot represents the isometrically mapped position.

![image](images/58fd72f4_image.png)

- $x = 1$

	![image](images/8ca37819_image.png)

- $y = 1$

	![image](images/a75cae77_image.png)

<br/>

## 🧊 Tiling isometrically

After understanding how to retrieve a position’s isometric counterpart, we may then use this logic to lay out each tile and form the world.


<div align="center">

This is an example of a tile asset that I drew:
<img src="images/9f34e38f_Untitled.png" width="200px"/>

</div>

A series of these tiles are aligned at fixed intervals, to form the world we see. 

Let’s say we want to render a 3x3 world. The basic way to go about it would be to increment x by an interval, and place an image at each increment.

For example, lets start by drawing our first row of squares, with each square being 20px in width. This means we have to increment by 20 to draw at the start of the next square:

<img src="images/c2e03ce0_image.png" width="300"/>
<br>
<img src="images/3143ae3a_image.png" width="300"/>
<br>
<img src="images/c53e604c_image.png" width="300"/>
<br>
<p>That was one row. Using the same steps, we then draw 2 more rows, but each row has a different starting y value. Just like $x$, we increment the $y$ by 20</p>

<img src="images/d6783d5d_image.png" width="300"/>

<img src="images/7e4056c2_image.png" width="300"/>

Our 3x3 world is complete. 

Let’s apply the same logic but this time, render isometrically. To breakdown the problem, let’s first map the points:

| **X Value** | **Image** |
|-------------|-----------|
| $x=0$     | ![image](images/db866bb5_image.png) |
| $x=20$    | ![image](images/1029af17_image.png) |
| $x=40$    | ![image](images/bd6f2f25_image.png) |

We have completed the first row. Next, we will increment the y, and draw the rows again

1. Cycle 2:

	<img src="images/50b4f6f2_image.png" width="500"/>

	1. Cycle 3:

	<img src="images/dccc6d51_image.png" width="500"/>

	<br/>

	With all positions mapped, we can then iterate. For each row, we start from the back instead of the front. Reversing the order of render is necessary to ensure that the images overlap properly.
	<span style='color:gray'>(you can try for yourself what happens when it is rendered starting from </span><span style='color:gray'>`(0,0)`</span><span style='color:gray'> instead of the last position)</span>

	<img src="images/2d1b7f7c_image.png" width="500"/>

	<img src="images/c5f6a5d5_image.png" width="500"/>

	<img src="images/526c2670_image.png" width="500"/>

	<img src="images/2d69ca45_image.png" width="500"/>

	<img src="images/82f7374d_image.png" width="500"/>

---

<br/>

Now, we have our 3x3 world

<div align="center"><img src="images/cbf8d737_image.png" alt="3x3 world" width="600"/></div>

Of course, we may want to populate it with other objects. Positioning objects atop an existing world will require the help of a third coordinate, z.

<div align="center"><img src="images/18e4e65f_image.png" alt="Objects on 3x3 world" width="600"/></td></div>

Once again, let’s refer to the way the x and y coordinate travel as they increase/decrease. When the x increases, the position travels towards the top right. When the y increases, the position travels to the bottom right. We will also introduce something known as the z coordinate. The z coordinate determines how high/low the object renders. Think of it as the true vertical axis (i.e. a substitute for the original y axis that no longer applies in an isometric world). The z coordinate is useful for rendering several layers of tiles. When the z coordinate increases, the position travels upwards.

<div align="center"><img src="images/2c3c865f_image.png" alt="Z coordinate example" width="300"/></div>
<div align="center"><img src="images/f28650a6_image.png" alt="Z coordinate example" width="300"/></div>
<p>(the z values here are only for demonstration purposes. They don’t represent the real height of the tiles)</p>

When it comes to rendering the world, there is a certain order to rendering the tiles. This order involves starting at the lowest/bottom-most layer (e.g.) floor, and working your way to the top (walls, player, troops). 

<img src="images/ebccf593_image.png" width="500"/>

In my game, there are only two of such layers: The floor layer and the wall layer.

1. Start by rendering first layer: Floor

	<img src="images/b257792a_image.png" width="500"/>

2. Increase Z value

3. Render layer 2: Objects

	<img src="images/2310900d_image.png" width="500"/>

Typically 2 layers is the easiest to work with. Having more layers introduces some additional complexities, however it is outside the scope of this article.

## 🎮 Isometric Assets

<img src="images/0249cd24_image.png" width="500"/>

<br/>

For the game to render properly in isometric format, graphics have to also adapt to such a format. This meant that the every asset had to abide by the following guidelines:

1. Tiles must be diagonally angled, with a diagonal edge that share the same steepness.

1. Tiles must be symmetrical

However, a property of isometric graphics is that it allows leeway in terms of sprite height and sprite visuals. Theoretically, I could make a sprite as tall as I wanted, and if rendered with the correct order, it would still display properly

<br/>

# 🏃 Animations

Two types of animation were applied to this project; Frame-by-frame animation and interpolation-based animation

## **Frame-by-Frame Animation Example**

In this method, every frame is individually drawn or created. When played in sequence, these frames create the illusion of movement. Below is an example of a sprite-sheet with frame-by-frame animation:

<div align="center">

![images/df7r99a-dd980548-1e81-4316-8d7d-d962f7d38b0a](images/0ad9d0cc_df7r99a-dd980548-1e81-4316-8d7d-d962f7d38b0a.gif)

By: [https://www.deviantart.com/spongedrew250/art/Sonic-Sonic-Running-Sprite-Sheet-920021662](images/https://www.deviantart.com/spongedrew250/art/Sonic-Sonic-Running-Sprite-Sheet-920021662)

</div>

## **Interpolation-Based Animation Examples**

This method uses algorithms and calculations to smoothly transition an object or character from one position, scale, or rotation to another over time.

<div align="center">

<img src="images/17dbb22c_ball.gif" width="500"/>

From: https://github.com/franciscop/ola

This is interpolation in its most basic representation. Using calculations, the position of the circle is moved towards its destination smoothly. This method is commonly used when dealing with rigs or 3D models that consist of joints and connected parts.

<br>

![ball](images/cd28515d_unnamed.gif)

Source: CyberLegends Animations

</div>

In frame-by-frame animation, movement is straightforward: I simply alternate between two frames every second.

However, using this method of animation for moving troops moving toward the player poses a problem. A preset frame-by-frame approach would be impractical because of the sheer number of frames required for all possible movement directions. Instead, I needed a dynamic way to smoothly glide enemies toward their target.

A common approach is to use what's known as **linear interpolation**, which uses the expression $A + (B - A) * t$ to calculate where a point in space should be between a start and end destination at a given time. However, there is a way to adapt this the linear interpolation function to be even simpler and easier to work with.


<br/>

**A surprisingly simple solution:**

Instead of tracking time explicitly, we can use a function that incrementally moves a value toward another:

```c
t_xy	interpolate(int value1, int value2, int slowness)
{
	return (t_xy){(value1 + (value2 - value1) / slowness)};
}
```

<br/>

This method can be extended to 2D positions as well:

```c
t_xy	interpolate(t_xy pos, t_xy pos2, int slowness)
{
	return ((t_xy){(pos.x + (pos2.x - pos.x) / slowness), (pos.y + (pos2.y - pos.y) / slowness)});
}
```

<br/>

**How This Works**
Unlike traditional interpolation that depends on a time parameter, this approach simply takes:
- The current position
- The target position
- A slowness factor
Each time the function is called, it returns a new position slightly closer to the target. To create movement, an object continuously updates its position using this function.

**Why This Method is Useful**
- Easing Effect – Unlike standard linear interpolation, this approach naturally slows down as it approaches the target. This form of movement is more natural to the eyes.
- No Time Tracking Required – Since the interpolation is based on the current position rather than a time factor, it works seamlessly for objects constantly chasing a moving target.

Below is an example using this function to smoothly interpolate an enemy's position as it travels to the player.

```c
#include <stdio.h>
#include <math.h>

typedef struct {
    float x, y;
} t_xy;

t_xy interpolate(t_xy pos, t_xy pos2, int slowness) {
    return (t_xy){pos.x + (pos2.x - pos.x) / slowness, pos.y + (pos2.y - pos.y) / slowness};
}

float distance(t_xy a, t_xy b) {
    return sqrtf((b.x - a.x) * (b.x - a.x) + (b.y - a.y) * (b.y - a.y));
}

int main() {
	t_xy enemy = {0, 0};
	t_xy player = {100, 50};
	int slowness = 10;

	printf("Enemy starting at (%.2f, %.2f)\n", enemy.x, enemy.y);

	// constantly reassigning its own value
	while (distance(enemy, player) > 0.1) {
		enemy = interpolate(enemy, player, slowness);
		printf("Enemy at (%.2f, %.2f)\n", enemy.x, enemy.y);
	}

	printf("Enemy reached the player!\n");
	return 0;
}
```

<br/>

---

# 📷 Camera tracking

Whenever we look at games that utilize camera tracking, there is a typical pattern where the player will always remain at the center of the screen, and the world around the player moves. This concept is how we execute the illusion of a camera; by not actually moving the player, but the objects around it.

This means that all objects in the world are rendered at their positions, however they must be offsetted using an offset coordinate value. 

This offset is determined by the camera, and the coordinates where the world is rendered has an inverse relationship to the position of the camera. 
- If you sit on a train moving to the north, you will see everything in the window moving southward. 
- By moving the camera to the right, the world move in the opposite direction, and vice versa. 

<br/>

To further understand, let's refer to this game world:   

<div align="center">
	<img src="images/Frame 34.png" alt="alt text" width="500"/>
</div>

<br>
Now I will introduce the camera. The camera captures a section of the screen. This section is whats shown in the viewport. Currently, the camera and the flower are located at (0, 0), hence both their centerpoints intersect.

<div align="center">
	<img src="images/Frame 46.png" alt="alt text" width="500"/>
</div>

<br>
The camera moves to the right. Notice that in the viewport, the flower is now at the left.
<div align="center">
	<img src="images/Frame 43.png" alt="alt text" width="500"/>
</div>

This is a demonstration of the inverse relationship between the camera's position, and that of the objects in the world. The same applies to the vertical axis as well.

For our purposes, camera just needs to be a coordinate. By keeping track of the camera's coordinates, we have a value that can be used to offset the world.

Let's say we render four objects, at the following positions:
- (300, 200)
- (600, 450)
- (1100, 300)
- (5, 800)


<div align="center">
<img src="images/Frame 36.png" alt="alt text" width="500"/>
</div>

<br>

By moving the camera right, the world moves left. Hence, if the camera moves by 300 pixels to the right, all images should be rendered 300 pixels to the left.
<div align="center">
<img src="images/Frame 47.png" alt="alt text" width="500"/>
</div>

To apply this to our game, we have to further process the positions of objects. We need to convert them to what I will term "viewport coordinates".

To covert the object to viewport coordinates, it's a simple matter of processing it using the following formula:

$$viewport\space pos = camera.pos - object.pos$$

The final questions to answer is "where should our camera be positioned at?" Well obviously it should focus on the player. 

<div align="center">
	<img src="images/Frame 44.png" alt="alt text" width="500"/>
</div>

At each game update, we may set the camera position to that of the player's, hence the player will always be at the center of the viewport. This makes it so that the player is central to the game, whilst the world around it moves.

But, can take this a step further than simply synchronizing the player and camera's position. We can get the camer to track the player smoothly, using the interpolation function from earlier to smoothen the camera movement. The  `destination` parameter is set to the current position of the player.

```c
// each game update
camera_pos = interpolate(camera_pos, player_pos, 4);
```

This will cause the camera to glide to the player gracefully.

<div align="center">
	<img src="images/Frame 45.png" alt="alt text" width="500"/>
</div>

# Summary and Conclusion

1. We keep track of the position of all objects in the world via a structs. These are their "true positions"
2. Each object's position can mapped isometrically by giving a relationship between the x and y, utilizing the iso_map function. This produces their isometric positions
3. We can introduce a camera to make the game feel more dynamic. In reality, the camera serves as a value that all objects in the world must be offseted by. This produces their viewport positions

<div align="center">
	<img src="images/Frame 48 (1)-1.png" alt="alt text" width="500"/>
</div>

---

<div align = "center">

## Follow me here

<img src="https://github.com/user-attachments/assets/dfc8e24f-122a-4d03-9f18-9ed4e464c435" width="300"> 
<img src="https://github.com/user-attachments/assets/6783bb52-7367-4f96-8fa5-e2919d633655" width="300">

</div>

---

<br/>

