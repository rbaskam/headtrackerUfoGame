<template>
    <div>
        <div class="row" id="holder">
        </div>
        <div class="row formHolder">
            <h4>Headtracker.js</h4>
            <p>Note: Broken on Safari at the moment.</p>
            <p>Simple Game, allow your webcam to see your face. Then move your head in the direction you want the space ship to move trying to hit the aliens.</p>
            
            <canvas id="inputCanvas" width="320" height="240" style="display:none"></canvas>
            <video id="inputVideo" autoplay loop playsinline></video>
        </div>
        
        <p>Feel free to use this or fork, available on <a href="https://github.com/rbaskam/headtrackerUfoGame" target="_blank">Github</a></a></p>
        <div v-if="!loading">
            <span><a href="/login">Login</a> or <a href="/register">Register</a> to add your scores.</span>
            <table class="table table-striped">
                <thead>
                <tr>
                    <th>User</th>
                    <th>Score</th>
                </tr>
                </thead>
                <tbody>
                    <tr v-for="score in scores">
                        <td></td>
                        <td></td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
    
</template>

<script>
    export default {
        mounted() {
            this.startHeadTracr();
            this.getGameScores();
        },
        data() {
            return {
               main: '',
               x: 0,
               y: 0,
               detected: false,
               speed: 1,
               category: 'high',
               scores: [], 
               loading: true
            }
        },

        methods: {
            getGameScores: function() {
                //Load you high scores from a route here and assign them to this.scores
                //The set this.loading to false to show the scores
            },
            startHeadTracr: function() {
                var videoInput = document.getElementById('inputVideo');
                var canvasInput = document.getElementById('inputCanvas');
                
                var htracker = new headtrackr.Tracker();
                htracker.init(videoInput, canvasInput);
                htracker.start();
                $(document).on('headtrackingEvent', e => {
                    // Get the amount the head moved left/right and up/down relative to the camera
                    var x = e.originalEvent.x;
                    var y = e.originalEvent.y / 2;
                    if (!this.detected) {
                        this.detectedFace(x, y)
                    }
                    this.movementDetector(x, y);
                })
            },
            detectedFace: function(x, y) {
                var self = this;
                self.detected = true;
                self.x = x;
                self.y = y;
                this.startGame();
                
            },
            movementDetector: function(x, y) {
                this.main(x, y);
                
            }, 
            startGame: function() {
                // Create the canvas
                var self=this;
                var canvas = document.createElement("canvas");
                var theDiv = document.getElementById("holder");
                var ctx = canvas.getContext("2d");
                canvas.width = 512;
                canvas.height = 480;
                theDiv.appendChild(canvas);

                // Game Vars
                var gameStarted = false;
                var isGameOver = false;

                // Background image
                var bgReady = false;
                var bgImage = new Image();
                bgImage.onload = function () {
                    bgReady = true;
                };
                bgImage.src = "https://github.com/rbaskam/headtrackerUfoGame/blob/master/images/background.png?raw=true";

                // Hero image
                var heroReady = false;
                var heroImage = new Image();
                heroImage.onload = function () {
                    heroReady = true;
                };
                heroImage.src = "https://github.com/rbaskam/headtrackerUfoGame/blob/master/images/ship.png?raw=true";

                // Monster image
                var monsterReady = false;
                var monsterImage = new Image();
                monsterImage.onload = function () {
                    monsterReady = true;
                };
                monsterImage.src = "https://github.com/rbaskam/headtrackerUfoGame/blob/master/images/alien.png?raw=true";

                // Game objects
                var hero = {
                    speed: 256 // movement in pixels per second
                };
                var monster = {};
                var monstersCaught = 0;

                // Reset the game when the player catches a monster
                var reset = function () {
                    self.speed = self.speed + 0.1;
                    if (!gameStarted) {
                        hero.x = canvas.width / 2;
                        hero.y = canvas.height / 2;
                        gameStarted = true;
                    }

                    // Throw the monster somewhere on the screen randomly
                    monster.x = 32 + (Math.random() * (canvas.width - 64));
                    monster.y = 32 + (Math.random() * (canvas.height - 64));
                };

                //Game Over
                var gameOver= function () {
                    monster = {}
                    isGameOver = true;
                    hero.x = canvas.width / 2;
                    hero.y = canvas.height / 2;
                    if (monstersCaught > 0) {
                        //Send to your route to save scores
                    }
                }

                // Update game objects
                var update = function (x,y) {
                    //Check if game is still running
                    if (isGameOver) {
                        return;
                    }
                    //Player Moved there head up
                    if (self.y < y) { 
                        hero.y -= self.speed;
                    }
                    //Player Moved there head down 
                    if (self.y > y) { 
                        hero.y += self.speed;
                    }
                    //Player Moved there head left
                    if (self.x > x) { 
                        hero.x -= self.speed;
                    }
                    //Player Moved there head right
                    if (self.x < x) { 
                        hero.x += self.speed;
                    }

                    //Have they touched the side of the screen
                    if ( hero.x >= (canvas.width + 16) || hero.x <= (0 - 16)) {
                        gameOver();
                    }

                    if ( hero.y >= (canvas.height + 16) || hero.y <= (0 - 16)) {
                        gameOver();
                    }

                    // Are the two characters touching?
                    if (
                        hero.x <= (monster.x + 32)
                        && monster.x <= (hero.x + 32)
                        && hero.y <= (monster.y + 32)
                        && monster.y <= (hero.y + 32)
                    ) {
                        ++monstersCaught;
                        reset();
                    }
                    
                };

                // Draw everything
                var render = function () {
                    if (bgReady) {
                        ctx.drawImage(bgImage, 0, 0);
                    }

                    if (heroReady) {
                        ctx.drawImage(heroImage, hero.x, hero.y);
                    }

                    if (monsterReady) {
                        ctx.drawImage(monsterImage, monster.x, monster.y);
                    }

                    // Score
                    ctx.fillStyle = "rgb(250, 250, 250)";
                    ctx.font = "24px Helvetica";
                    ctx.textAlign = "left";
                    ctx.textBaseline = "top";
                    if (isGameOver) {
                        ctx.fillText("Game Over: Total Aliens caught: " + monstersCaught, 32, 32);
                    } else {
                        ctx.fillText("Aliens caught: " + monstersCaught, 32, 32);
                    }
                };

                // The main game loop
                self.main = function (x, y) {
                    update(x, y);
                    render();
                };

                // Let's play this game!
                reset();
            }
        }
    }
</script>
