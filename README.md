# culminating

## Mainsketch
// module aliases
var Engine = Matter.Engine,
	Render = Matter.Render,
	Runner = Matter.Runner,
	Composites = Matter.Composites,
	Events = Matter.Events,
	Constraint = Matter.Constraint,
	MouseConstraint = Matter.MouseConstraint,
	Mouse = Matter.Mouse,
	Body = Matter.Body,
	Composite = Matter.Composite,
	Bodies = Matter.Bodies,
	World = Matter.world

var engine = Engine.create();
var bullet
var player2aim

//let firstblock;
let player2;
let blocks = []
let ground;
let theimage;

function preload() {
	img = loadImage('lava.avif')
}


function setup() {
	//collection of blocks     we need an array
	leftWallX = windowWidth / 2 - 200
	rightWallX = windowWidth / 2 + 200
  scorelineY = 300
	
	blockxposition = windowWidth / 2
	blockyposition = 100
	// create an engine

	// create a renderer
	var render = Render.create({
		element: document.body,
		engine: engine,
		options: {
			width: document.body.clientWidth,
			height: document.body.clientHeight,
			wireframes: false,
			background: ('orange')
		}
	});
	createCanvas(windowWidth, windowHeight, render.canvas)

	// create objects and a ground
	player2 = Bodies.circle(200, 200, 20, {
		friction: 2,
		restitution: 0.7
	})
	ground = Bodies.rectangle(800, windowHeight, 100000, 60, {
		isStatic: true
	});
	var roof = Bodies.rectangle(800, 0, 100000, 60, {
		isStatic: true
	});

	//Use the following    Bodies.fromVertices

	// add all of the bodies to the world
	Composite.add(engine.world, [player2, ground, roof]);

	// run the renderer
	Render.run(render);

	// create runner
	var runner = Runner.create();

	// run the engine
	Runner.run(runner, engine);
}

function draw() {
	if (clickcount % 2 == 0)
		if (!pause) {}
	//background(img)
	screenhop()
	borders()

	timer()
	fill("black")
	textAlign(CENTER);
	textSize(50)
	text(time, windowWidth - 70, 80)

	if (playerBullet)
		playerBullet.update()
	checkwinner()
	if (Player2wins > 0) {
		updatewinner()
	}
	
	checkwinner()
	if (player1wins > 0)
		updatewinner()
	if (who === 1) {
		who++
		checkforblock()
	}
	
	// if the current block exists
	if (playerBlocks) {
		// check if it collides with the ground
		if (playerBlocks.collides(ground)) {
			// set flag to spawn a new block
			who--
		}
		else {
			//go through each block in the blocks array and check if the current block has collided with it
			// and if so spawn another block
			let n = blocks.length

			for (let i = 0; i < n; i++) {
				let block = blocks[i]
				if (playerBlocks.collides(block.body)) {
					who--

					break
				}
			}
		}
	}
}

## Variables
let blockxposition = 0
let blockyposition = 0


let who = 1
let h = 10
let w = 10
let r = 20
let win = 1
let Player2 = "PLAYER 2 WINS"
let player1 = "PLAYER 1 WINS"
let Player2wins = 0
let player1wins = 0
let menuY = 25
let menuX = 25
let menuSize = 50
let clickcount = 0
let pause = false

## keypress
function keyPressed() {
	if (keyCode === ENTER) pause = !pause; {
		//Cancels all game inputs when pressed and resumes game functions after its pressed
		if (!pause) {


			//if up arrow is pressed, then the ball goes up
			if (keyCode == UP_ARROW) {
				var position1 = player2.position
				var direction1 = createVector(0, -0.06)
				Matter.Body.applyForce(player2, position1, direction1)
			}

			//if down is pressed, then the ball goes down
			if (keyCode == DOWN_ARROW) {
				var position2 = player2.position
				var direction2 = createVector(0, 0.05)
				Matter.Body.applyForce(player2, position2, direction2)
			}

			//if left arrow if pressed, then ball goes left
			if (keyCode == LEFT_ARROW) {
				var position3 = player2.position
				var direction3 = createVector(-0.07, 0)
				Matter.Body.applyForce(player2, position3, direction3)
			}

			//if right arrow is pressed, then ball goes right
			if (keyCode == RIGHT_ARROW) {
				var position4 = player2.position
				var direction4 = createVector(0.07, 0)
				Matter.Body.applyForce(player2, position4, direction4)
			}
//if d is pressed, playerBlocks is translated to the right
			if (key == 'd') {
				Body.translate(playerBlocks.body, {
					x: 40,
					y: 0
				})
			}
//if a is pressed, playerBlocks is translated to the left
			if (key == 'a') {
				Body.translate(playerBlocks.body, {
					x: -40,
					y: 0
				})
			}
		}
	}
}

## screenhop
function screenhop() {
	//if player2 x position is smaller than 0, then body will be translated to the right side of the screen
	if (player2.position.x < 0)
		Body.translate(player2, {
			x: windowWidth,
			y: 0
		})
	
//if player2 x position is greater than 0, then body will be translated to the left side of the screen
	if (player2.position.x > windowWidth)
		Body.translate(player2, {
			x: -windowWidth,
			y: 0
		})
}

## player borders
let leftWallX, rightWallX

function borders() {
	line(leftWallX, windowHeight, windowWidth / 2 - 200, 0)
	line(rightWallX, windowHeight, windowWidth / 2 + 200, 0)
 line(leftWallX, scorelineY, rightWallX, scorelineY)

	//propells player2 back after charging into the border
	if (player2.position.x > leftWallX && player2.velocity.x > 0 && player2.position.x < rightWallX) {
		let diff = player2.position.x - leftWallX + r
		var stop = createVector(-0.039, 0)
		Matter.Body.applyForce(player2, player2.position, stop)
		Body.translate(player2, {
			x: -diff,
			y: 0
		})

	} else if (player2.position.x < rightWallX && player2.velocity.x < 0 && player2.position.x > leftWallX) {
		let diff = rightWallX - player2.position.x + r
		var stop = createVector(0.039, 0)
		Matter.Body.applyForce(player2, player2.position, stop)
		Body.translate(player2, {
			x: diff,
			y: 0
		})
	}
}

## player 2 bullets
let playerBullet


// Bullet properties
class Bullet {
	constructor(force = 0.015, mass = 0.09, radius = 10, colour = 'blue') {
		this.radius = radius
		this.fireTime = 0
		this.force = force
		this.mass = mass
	}

	fire(position, direction) {
		this.direction = direction
		this.body = Bodies.circle(position.x, position.y, this.radius, {
			render: {
				fillStyle: this.colour,
				strokeStyle: 'black',
				lineWidth: 1
			}
		})
		Body.setMass(this.body, this.mass);

		// set the bullet position
		Composite.add(engine.world, [this.body]);

		// set the force in the direction
		this.direction.setMag(this.force)

		// apply the force to the bullet
		Body.applyForce(this.body, this.body.position, this.direction)
		this.fireTime = millis()
	}

	update() {
		var now = millis()
		if (this.body && this.fireTime > 0 && now - this.fireTime > 2000) {
			Composite.remove(engine.world, this.body)
			
		}
	}
}

//if mouse is clicked, then bullets are firsed to the mouse axis
function mouseClicked() {
	playerBullet = new Bullet()
	v1 = createVector(player2.position.x, player2.position.y)
	v2 = createVector(mouseX, mouseY)
	v3 = v2.sub(v1)
	playerBullet.fire(v1, v3)
}

## timer
//use millis which is a function
let lastime = 0
let time = 120
let s = 1

function timer() {
	let now = millis()
	if (now - lastime > 1000) {
		lastime = now
		time-= s
	}
	//if timer reaches 0, then timer stops
	if (time <=0){
		s = 0
	}
}

## win
function checkwinner() {
	//if timer reaches 0, player 2 wins 
	if (time <= 0) {
		Player2wins++
	}
	//if blocks reach a certain height, player 1 will win // in progress
	else if (playerBlocks && playerBlocks.body.position.y > 800){
		print(playerBlocks)
		player1wins++
	}
}

function updatewinner() {
	if (Player2wins > 0) {
		fill("black")
		textAlign(CENTER);
		textSize(70)
		text(Player2, windowWidth / 2, windowHeight / 2)
	}
	else if (player1wins > 0) {
		fill("black")
		textAlign(CENTER);
		textSize(70)
		text(player1, windowWidth / 2, windowHeight / 2)
	}
}

## player 1 blocks
let playerBlocks

class smashboy {

	constructor(name, sides, sidelength, mass = 999999999999) {
		this.name = name
		this.sides = sides
		this.sidelength = sidelength
		this.mass = mass
	}

	place(position) {
		this.body = Bodies.rectangle(position.x, position.y, this.sides, this.sidelength, {
			render: {
				fillStyle: this.colour,
				strokeStyle: 'black',
				lineWidth: 2
			}
		})
		Composite.add(engine.world, [this.body]);
	}
	collides(body) {
		if (this.body && Matter.Collision.collides(this.body, body)) {
			return true
		} else return false
	}

}


class hero extends smashboy {

	constructor(sidelength) {
		super(40);
		this.name = 'square';
		this.sides = 4;
	}
}

const newsquare = new smashboy('hi', 5, 6);
const newhero = new hero(5);


function checkforblock() {

	if (playerBlocks) {
		//if conditions for player 1 are met, then player 1 wins
		if (playerBlocks.body.position.y < 300 && playerBlocks.body.speed){
				player1wins++
				}
		blocks.push(playerBlocks)
	}
	playerBlocks = new smashboy("smashboy", 40, 40)
	playerBlocks.place(createVector(blockxposition, blockyposition))
}
