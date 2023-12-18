![Snake](https://github.com/QuinnChrest/Snake/assets/30156594/80d392c4-423a-4515-adca-d687c9c56560)

I wanted to challenge myself to create a simple game in Javascript. First I needed to decide what libraries I was going to use. I landed on [PixiJS](https://pixijs.com/) to handle rendering and game ticker. Since the plan is to make a simple game I landed on recreating Snake. Solving how the snake moves across the board with the elements of its body following its head interested me. Below I will touch on some of the basic functionality of the game.

## Handling Movement

Using javascript we can capture keyboard key presses using the key-down event listener.

`window.addEventListener("keydown", keyDown);`

Now we can capture what is pressed on the keyboard and set a local move direction variable based on what is pressed.

```
function keyDown(e) {
  // based on what key is pressed set the direction to move the snake head
  switch (e.keyCode) {
    case 87:
    case 38:
      if (
        snake.length == 1 ||
        (snake.length > 1 && snake[1].y != snake[0].y - 50)
      ) {
        moveDir = direction.Up;
      }
      break;
    case 40:
    case 83:
      if (
        snake.length == 1 ||
        (snake.length > 1 && snake[1].y != snake[0].y + 50)
      ) {
        moveDir = direction.Down;
      }
      break;
    case 65:
    case 37:
      if (
        snake.length == 1 ||
        (snake.length > 1 && snake[1].x != snake[0].x - 50)
      ) {
        moveDir = direction.Left;
      }
      break;
    case 39:
    case 68:
      if (
        snake.length == 1 ||
        (snake.length > 1 && snake[1].x != snake[0].x + 50)
      ) {
        moveDir = direction.Right;
      }
      break;
  }
  
  // if we are in a game over state then pressing any key will reset the game
  if (gameOver) {
    cleanUp();
  
    setInitialValues();
  
    gameOver = false;
  }
}
```
Something to note here is you can see multiple checks similar to `snake.length == 1 || (snake.length > 1 && snake[1].y != snake[0].y - 50)`. This is so that when the snake has more than one body part it can't move backwards into itself.

So now that a move direction has been specified to the game then on the game loop the snake will auto move in that direction.

```
function moveHead() {
  // store x y coords of the snake head before moving
  let x = snake[0].x;
  let y = snake[0].y;

  // based on the current move direction move the head in that direction
  if (moveDir == direction.Up && snake[0].y > 0) {
    snake[0].y -= 50;
  } else if (moveDir == direction.Down && snake[0].y < 550) {
    snake[0].y += 50;
  } else if (moveDir == direction.Left && snake[0].x > 0) {
    snake[0].x -= 50;
  } else if (moveDir == direction.Right && snake[0].x < 550) {
    snake[0].x += 50;
  } else if (moveDir != null) {
    endGame();
  }

  // if snake has a body then move the body as well
  if (snake.length > 1 && !gameOver) {
    moveBody(1, x, y);
  }

  // if snake head collides with pellet then move pellet and add to the snakes body
  if (snake[0].x == pellet.x && snake[0].y == pellet.y) {
    updateScore();
    placePellet();
    createSnakeBody(snake[snake.length - 1].x, snake[snake.length - 1].y);
  }
}
```
Here we are checking to make sure that the snake isn't running into the wall before moving itself. Collision with the wall and the pellet is handled in this portion of the code since we are already checking the snake heads position.

Once the head moves the body can recursively follow.

```
function moveBody(index, x, y) {
  // store old x y coords for next snake piece
  let old_x = snake[index].x;
  let old_y = snake[index].y;

  // move snake body at index provided to x y coordinate provided
  snake[index].x = x;
  snake[index].y = y;

  // check if snake head collided with body
  if (snake[0].x == snake[index].x && snake[0].y == snake[index].y) {
    endGame();
  }

  if (++index < snake.length) {
    // move next part of the body if one exists
    moveBody(index, old_x, old_y);
  }
}
```
I thought that the snake movement was going to be more complicated but then I realized I could just have each part of the body follow the body part in front of it in the array. 🤷

## Scoring

As stated above the collision with the pellet is handled during movement. When collecting the pellet it will increment the score and randomly move itself to a new location that is not currently being occupied. Spiced up the scoring code by making pellets worth more points based on how close the wall they are. This value is multiplied by the snake's current length.

![image](https://github.com/QuinnChrest/Snake/assets/30156594/26b44095-a487-4dbd-a060-8974f0eadd99)


```
function updateScore() {
  if (
    pellet.y >= 250 &&
    pellet.y <= 300 &&
    pellet.x >= 250 &&
    pellet.x <= 300
  ) {
    score += snake.length * 1;
  } else if (
    pellet.y >= 200 &&
    pellet.y <= 350 &&
    pellet.x >= 200 &&
    pellet.x <= 350
  ) {
    score += snake.length * 2;
  } else if (
    pellet.y >= 150 &&
    pellet.y <= 400 &&
    pellet.x >= 150 &&
    pellet.x <= 400
  ) {
    score += snake.length * 3;
  } else if (
    pellet.y >= 100 &&
    pellet.y <= 450 &&
    pellet.x >= 100 &&
    pellet.x <= 450
  ) {
    score += snake.length * 4;
  } else if (
    pellet.y >= 50 &&
    pellet.y <= 500 &&
    pellet.x >= 50 &&
    pellet.x <= 500
  ) {
    score += snake.length * 5;
  } else if (
    pellet.y >= 0 &&
    pellet.y <= 550 &&
    pellet.x >= 0 &&
    pellet.x <= 550
  ) {
    score += snake.length * 6;
  }

  window.document.querySelector(".score").innerHTML = score;
}
```
