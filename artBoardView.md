# ArtBoard View

ArtBoard View is responsible for rendering the artBoards that the user can draw on. The entry point for this view is the `ArtBoard.tsx` file. The `ArtBoard.ts` component calls this view to render the artBoard.<br />

The files correspnding to this view are:
<ul>
<li> ArtBoard.tsx
<li> ArtBoardHandler.tsx
<li> ArtBoardSketch.tsx
<li> ArtBoardTurtle.tsx
</ul>

---
&nbsp;
## Structure
The artBoard is divided into 2 types of canvas. ArtBoardSketch handles the drawing functionality, and ArtBoardTurtle renders the different turtles. If there are N canvases, ArtBoardSketch is called N times and artBoardTurtle is called 1 time. <br />

![](images/artBoardStructure.png) 

---
&nbsp;
## ArtBoard.tsx
It is the entry point for the view. It contains the props to move each turtle on the artBoard. It also exposes the setting of each turtle. These settings define how the turtle will draw on the artBoard. The artBoard 

### Functions to move the turtle: 

```
moveTurtleInArc()
MoveTurtleForward()
RotateTurtle()
```

To specify the turtle that should move can be done by specifying the id of the turtle you want to move to the `selectedTurtle` variable.

### Turtle Settings:

Parameters | type | description
--- | --- | ---
arcRadius | number | The radius of the arc the turtle will draw
arcAngle | number | The angle of the turtle will rotate while moving on an arc
rotateAngle | number | The angle the turtle will rotate about its centre
moveDirection | string | The direction the turtle will move in (Can be forward or back)
distance | number | The distance the turtle will move either forward or back
sleepTime | number | This donates the delay when a turtle rotates by an angle. The higher the value, the slower does the turtle move.
moveSleepTime | number | This donates the delay when either turtle move in a forward, back or an arc. The higher the value, the slower does the turtle move.

ArtBoardList is fetched from context API and contains the list of all the artBoards that are being rendered.

## ArtBoardHandler.tsx
ArtBoardHandler is an intermediate file between artBoard.tsx and artBoardSketch.tsx to set the move, rotate and arc functionalities different for each canvas. It also compares the turtle id with the selectedTurtles id and sends a signal to artBoardSketch.tsx to move the selectedTurtle only. <br />

Boolean variables are used to send a signal to artBoardSketch.tsx to move the selectedTurtle only. This is done using the `useEffect` hook.

## ArtBoardSketch.tsx

Contains the functionality of the turtle to move the turtle forward, backward, in an arc or rotate the turtle. It does not render the turtle on the canvas; it just draws the line for each turtle. `ArtBoardDraw.ts` model is called to specify the thickness and colour of the line drawn.
P5 instance is created in each file and is used to draw the lines of the turtle. It creates a canvas as a p5 element.<br /> There are two important functions setup and draw. The code inside the draw() function runs continuously from top to bottom until the program is stopped. The setup function runs only once in the beginning.

Each functionality is divided into 2 parts, and there are 3 operations that we can do on a turtle.

<ol>
<li> MakeArc
<li> RotateTurtle 
<li> Move Forward/Backward
</ol>

### MakeArc
The first part of the function defines the angle by which the turtle will rotate and the radius of the arc.
```
async function makeArc(angle, radius) {

      for (let i = 0; i < angle; i++) {
        await sleep(turtleSettings.moveSleepTime);
        makeArcSteps(i, radius);
      }
    }
```

The for loops runs equal to the angle we want to rotate the turtle. The sleep function is used to dela the turtle to rotate and the makeArcSteps changes the position of the turtle step by step (according to the radius) so it feels the turtle is rotating in an arc.
```
function makeArcSteps(i, radius) {
      let initialX = currentTurtle.getTurtleX();
      let initialY = currentTurtle.getTurtleY();

      let finalX = initialX + radius * sketch.cos(currentTurtle.getTurtleAngle() + 1);
      let finalY = initialY - radius * sketch.sin(currentTurtle.getTurtleAngle() + 1);

      sketch.line(initialX, initialY, finalX, finalY);

      currentTurtle.setTurtleX(finalX);
      currentTurtle.setTurtleY(finalY);

      currentTurtle.setTurtleAngle(currentTurtle.getTurtleAngle() + 1);
    }
```

This same process is repeated for moving turtle forward/backwards and rotating turtle.


## ArtBoardTurtle.tsx
ArtBoardTurtle.tsx is responsible for rendering the turtle and moving it when we draw on the artBoard. It also handles the drag and drops functionality of the turtle. All the different turtles are rendered on the same canvas, and each turtle is associated with a canvas. The canvas and the turtle share the same unique id.

Turtle class is used to store the turtle's position and angle, and the `render` functions handle all the movement and rotation of the turtle.

The draw function in p5 runs in an infinite loop.
```
sketch.draw = () => {
    sketch.clear();
    artBoardList.map((artboard) => artboard._turtle.render(sketch));
  };
```
The canvas is cleared at every interval to remove the turtle from the previous position. ArtBoardList is iterated over, and render called on each turtle to updates its position and angle.

---
&nbsp;
### Drag and Drop
P5 provides us with 2 functions `mousePressed()` and `mouseDragged()`.These functions are used to handle the drag and drop of the turtle.

The mousePressed() function is called once after every time a mouse button is pressed. The function checks if the mouse is over a turtle, and if it is, it sets the active variable of the turtle to true. This also checks that if the mouse is moving, it will not set the active variable to true.<br/>
```
sketch.mousePressed = () => {
      for (let i = 0; i < artBoardList.length; i++) {
        let turtle = artBoardList[i]._turtle,
          distance = sketch.dist(sketch.mouseX, sketch.mouseY, turtle._turtleX, turtle._turtleY);
        if (!turtle.getIsMoving()) {
          if (distance < 30) {
            turtle._active = true;
          } else {
            turtle._active = false;
          }
        }
      }
  };
```
The mouseDragged() function is called once the mouse moves and a mouse button is pressed. It starts searching from the top most turtle, and if any of the turtles is active, it will update the turtle's position according to the mouse position.<br/>
```
  sketch.mouseDragged = () => {
      for (let i = artBoardList.length - 1; i >= 0; i--) {
        let turtle = artBoardList[i]._turtle;
        if (turtle._active) {
          turtle._turtleX = sketch.mouseX;
          turtle._turtleY = sketch.mouseY;
          break;
        }
      }
};
```