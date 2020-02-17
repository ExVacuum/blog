---
title: Experimental Game Mechanics in Java - Transparent Windows
layout: default
nav_order: 1
permalink: /programming/trans-dialogs
parent: Programming
authors: ['exvacuum']
---

# Experimental Game Mechanics in Java - Transparent Windows

## February 6, 2020

![headerimg]

[headerimg]:../../assets/images/floatingwindowheader.png

**Example of a "floating" player object.**

Recently, while working on an experimental game prototype, the following thought occurred to me: "Wouldn't it be cool if the player could walk right out of the window bounds?"
After some trial and error, I found a solution that works quite well for myself: creating a distinct window for the player object that could give the appearance of levitation
in traditionally unplayable space. I've created this post as a means to save others some frustration, as well as just to showcase something cool I figured out.

> **Note: This tutorial assumes a basic understanding of Java AWT/Swing. This mechanic is only confirmed to work on Windows 10, alternative solutions may be necessary depending on the OS.**

There are multiple ways to go about achieving the desired result. That being said, I want to make sure that this solution is as adaptable as possible for a wide variety of applications.
In this case, I have decided to create a single `JFrame` window that will act as the parent. We can extend `JFrame` in the main class of our program, or initialize one later on.
Now, our program will look something like this:

```java

/*I've decided to extend JFrame, 
however this is up to you.*/
class Main extends JFrame{ 

    //Main drawing panel
    JPanel drawingPanel;

    public static void main(String[] args){
        
        //Initialize the game
        init();

        //Main game loop
        while(true){
            step(); //All game logic
            draw(); //All drawing
        }
    }
        
    void init(){

        //Setup window here...
    
        //Add drawing panel to frame
        drawingPanel = new MPanel();
        add(drawingPanel);
    }

    void step(){
        
        //Game logic

    }

    void draw(){

        //Repaint main panel
        drawingPanel.repaint();
    }
    
    //Custom JPanel class for custom painting
    class MPanel extends JPanel{
       @Override
       public static void paintComponent(Graphics g){

            //Drawing code goes here...
        }    
    }
}
```

We will come back to our `init()` method later on, once our player class has been created. We'll now create this new `Player` class:

```java

/*Our player class will extend JDialog in order to exist in a separate component,
and it implements FocusListener in order to maintain priority over the background window.*/
class Player extends JDialog implements FocusListener{
    
    //JPanel for player drawing
    public JPanel pane;

    public Player(Frame owner){
        super(owner);
        init();
    }

    void init(){

        setVisible(false); // Dialog should be hidden before modification

        //Define window size and position here

        //Important lines here:
        setUndecorated(true); // Removes the "bars" on the dialog.
        setBackground(new Color(0,0,0,0)); //Sets the background of the dialog to transparent
        pane = new PlayerPane(); // Create drawing panel
        pane.setOpaque(false); //Remove panel background
        add(pane); // Add panel
        addFocusListener(this); // Enable listening for focus events
        setVisible(true); // Restore visibility
        requestFocus(); // Bring this dialog to the front.
    }

    public void step(){
        
        //Player Logic

    }
    
    @Override
    public void focusGained(FocusEvent focusEvent) {
    }

    //We must ensure that the player retains focus if the background window is clicked.
    @Override
    public void focusLost(FocusEvent focusEvent) {
        requestFocus();
    }

    public void draw(){
        pane.repaint();
    }   

    class PlayerPane extends JPanel{
    
            @Override
            public void paintComponent(Graphics g){

                //Drawing for inside of player goes here
            }
    }

}

```

Now that our `Player` class is ready, we can add it to the main class of our program:

```java

Player player;

void init(){

    //Setup window here...     

    // Add drawing panel to frame
    drawingPanel = new MPanel();
    add(drawingPanel);

    //Add player
    player = new Player(this);
}

void step(){
    
    //Game logic

    //Player logic
    player.step();
}

void draw(){

    //Repaint main panel
    drawingPanel.repaint();

    //Draw player
    player.draw();
}
```

Now this should result in a program that looks something like this:

![result]

[result]:../../assets/images/result.png

And there you have it! A bizarre little mind-bending effect that can add an element of surreal to any game.