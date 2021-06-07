<!--*************************************************************************************************************

    TEMPLATE/HTML AREA

*************************************************************************************************************-->
<template>
    <div>
        <div class="container">
            <div class="totalPoints"><!-- GAME TOTAL POINTS -->
                <img src="../assets/b_blue.png" width="40px" height="40px">
                {{ game.totalPoints }} <sup class="extraPoints">next +{{ bubbles.extraPoints }}</sup>
            </div>
            <div class="availableBubbles"><!-- AVAILABLE BUBBLES FOR THE PLAYER -->
                <img src="../assets/cannon.png" width="60px" height="50px">
                <img src="../assets/b_blue.png" width="20px" height="20px" v-for="index in (bubbles.maxBlueBubbles - bubbles.list.length)" :key="index">
                {{ bubbles.maxBlueBubbles - bubbles.list.length }}
            </div>
        </div>
        <img :src="bubbles.bRed.style.src" :style="bubbles.bRed.style" v-if="!bubbles.isFirstBubble"><!-- RED BUBLE -->
        <img :src="bubble.style.src" v-for="bubble of bubbles.list" :style="bubble.style" :key="bubble.id"><!-- BLUE BUBLES -->
    </div>
</template>
<!--*************************************************************************************************************

     JAVASCRIPT/OBJECTS AREA

*************************************************************************************************************-->
<script>
export default {
    name: 'RedBubble',
    data: function() {
        return {
            game: { // GAME GENERAL PROPERTIES
                windowWidth  : parseInt(window.innerWidth ),
                windowHeight : parseInt(window.innerHeight),
                totalPoints  : 0,
                audio: {
                    soundtrack: new Audio(require('../assets/audio/back.mp3'))
                }
            },
            bubbles: { // BUBBLES GENERAL PROPERTIES
                list            : [],         // LIST OF BLUE BUBBLES
                currentID       : 0,          // USED FOR THE :key BINDING PROPERTY NEEDED FOR THE v-for DIRECTIVE
                intervalIDs     : [],         // LIST OF IDs USED BY THE clearInterval() METHOD
                bRed            : undefined,  // RED BUBBLE
                growthRate      : 2,          // RED BUBBLE's GROWTH RATE USED EVERYTIME A BLUE ONE IS HIT
                shrinkRate      : 0,          // BLUE BUBBLE's SHRINK RATE USED EVERYTIME A BLUE ONE IS HIT
                step            : 1,          // BLUE BUBBLE'S SPEED
                isFirstBubble   : true,       // FIRST BUBBLE IS ALWAYS THE RED ONE
                maxBlueBubbles  : 5,          // MAX BUBBLES THAT CAN BE TRIGGERED BY THE PLAYER
                extraPoints     : 1,          // EVERYTIME THE RED BUBBLE HITS A BLUE ONE, extraPoints ADDS +1
                audio: {
                    pops: [
                        new Audio(require('../assets/audio/pop1.wav')),
                        new Audio(require('../assets/audio/pop3.wav')),
                        new Audio(require('../assets/audio/pop2.wav')),
                    ],
                    newBubble: new Audio(require('../assets/audio/newBubble.wav')),
                    redBubble: new Audio(require('../assets/audio/redBubble.wav'))
                },
                const: {
                    initialWidth   : 50,    // INITIAL VALUE FOR EVERY BUBBLE
                    initialHeight  : 50,    // INITIAL VALUE FOR EVERY BUBBLE
                    margin         : 30     // USED TO AVOID THE BUBBLE HIT THE BOUNDARIES OF THE WINDOW
                }
            }
        }
    },
    created: function() {
        window.onclick = this.createNewBubble
        setInterval(() => { this.checkCollisions() }, 1) // GAME MAIN LOOP. EXECUTES EVERY 1 MILLISECOND
    },
    methods: {
        /*****************************************************************************************************
         * createNewBubble(mouseEvent): CREATES A NEW BUBBLE WITH COORDS [this.generateInitialCoords(mouseEvent)] 
         * RANDOM DIRECTION [rightWay, bottomWay], RANDOM SPEED [rightStep, bottomStep]
         * AND THEN CALLS THE moveBubble(bubble) METHOD
         *****************************************************************************************************/
        createNewBubble: function(mouseEvent) {
            if(this.bubbles.list.length == this.bubbles.maxBlueBubbles)
                return
            let coords    = this.generateInitialCoords(mouseEvent)
            let newBubble =  {
                style: {
                    width      : (this.bubbles.const.initialWidth  - this.bubbles.shrinkRate) + 'px',
                    height     : (this.bubbles.const.initialHeight - this.bubbles.shrinkRate) + 'px',
                    position   : 'absolute',
                    transform  : 'rotate(0deg)',
                    left       : coords.x +'px',
                    top        : coords.y +'px',
                    src        : require('../assets/b_blue.png')
                },
                id         : this.bubbles.currentID,
                rightWay   : this.generateRandomDirection(),
                bottomWay  : this.generateRandomDirection(),
                rightStep  : Math.floor(Math.random() * this.bubbles.step) + 1,
                bottomStep : Math.floor(Math.random() * this.bubbles.step) + 1,
                rotateDeg  : 0
            }
            this.bubbles.currentID++
            this.bubbles.extraPoints     = 1      // RESET extraPoints EVERYTIME THE PLAYER CLICKS
            this.moveBubble(newBubble)
        },
        /****************************************************************************************************
         * generateInitialCoords(mouseEvent): HAS SOME RANDOM FACTOR TO DECIDE THE INITIAL { x, y } 
         *                                    WICH WILL BE RETURNED
         ****************************************************************************************************/
        generateInitialCoords: function(mouseEvent) {
            let deltaX = Math.floor(Math.random() * 30) + 100
            let deltaY = Math.floor(Math.random() * 30) + 100
            let x      = (mouseEvent.clientX < this.game.windowWidth /2)? (mouseEvent.clientX + deltaX) : (mouseEvent.clientX - deltaX)
            let y      = (mouseEvent.clientY < this.game.windowHeight/2)? (mouseEvent.clientY + deltaY) : (mouseEvent.clientY - deltaY)
            return { x, y }
        },
        /***********************************************************************************************************
         * moveBubble(bubble): MOVES THE bubble EVERY 1ms TO THE SAME DIRECTION THAT THE bubble HAS.
         * ROTATES THE IMAGE USING THE SAME DIRECTION OF THE bubble
         * CALLS validateCoords(bubble) TO CORRECT THE DIRECTION IF NEEDED
         * ASSINGS THE redBubble [ONLY THE FIRST TIME THIS MEHOD IS CALLED]
         * ADDS THE BUBBLES TO THE GLOBAL ARRAY bubbles.list FOR LATER COLLISION VALIDATIONS
         * ADDS THE setInterval's id OF EVERY BUBBLE TO THE GLOBAL ARRAY bubbles.intervalIDs
         *      FOR LATER COLLISION VALIDATIONS [WILL BE NEEDED TO CANCEL THE INTERVAL WITH cancelInterval() METHOD]
         ************************************************************************************************************/
        moveBubble: function(bubble) {
            let id = setInterval( () => {
                let leftFactor          = (bubble.rightWay )? 1 : -1
                let topFactor           = (bubble.bottomWay)? 1 : -1
                let newLeft             = parseInt(bubble.style.left) + (bubble.rightStep  * leftFactor)
                let newTop              = parseInt(bubble.style.top ) + (bubble.bottomStep * topFactor)
                bubble.rotateDeg       += (bubble.rightStep * leftFactor)
                bubble.style.transform  = 'rotate(' + bubble.rotateDeg + 'deg)'
                bubble.style.left       = newLeft + 'px'
                bubble.style.top        = newTop  + 'px'
                this.validateCoords(bubble)
            }, 1)

            if(this.bubbles.isFirstBubble) {
                bubble.style.src           = require('../assets/b_red.png')
                this.bubbles.bRed          = bubble
                this.bubbles.isFirstBubble = false
                this.game.audio.soundtrack.volume = 0.3
                this.game.audio.soundtrack.play()
                this.bubbles.audio.redBubble.play()
            } else {
                this.bubbles.audio.newBubble.play()
                this.bubbles.list.push(bubble)
                this.bubbles.intervalIDs.push(id)
            }
        },
        /***********************************************************************************************************
         * validateCoords(bubble): VALIDATES IF THE BUBBLE EXCEEDS ANY BOUNDARY AND CORRECT THE DIRECTION
         *      > IF THE x (CSS/left) EXCEEDS THE LEFT OR RIGHT BOUNDATY, WILL CHANGE THE DIRECTION TO THE OPPOSITE
         *      > IF THE y (CSS/top ) EXCEEDS THE TOP OR BOTTOM BOUNDATY, WILL CHANGE THE DIRECTION TO THE OPPOSITE
         ***********************************************************************************************************/
        validateCoords: function(bubble) {
            let maxX = (this.game.windowWidth  - (parseInt(bubble.style.width ) + this.bubbles.const.margin))
            let maxY = (this.game.windowHeight - (parseInt(bubble.style.height) + this.bubbles.const.margin))
            let x    = parseInt(bubble.style.left)
            let y    = parseInt(bubble.style.top )
            if(x <= this.bubbles.const.margin) {
                x = this.bubbles.const.margin
                bubble.rightWay = true
            } else if(x > maxX) {
                x = maxX
                bubble.rightWay = false
            } else {
                if(bubble == this.bubbles.bRed && this.generateDirectionChange()) {
                    bubble.rightWay = !bubble.rightWay
                }
            }
            if(y <= this.bubbles.const.margin) {
                y = this.bubbles.const.margin
                bubble.bottomWay = true
            } else if(y > maxY) {
                y = maxY
                bubble.bottomWay = false
            } else {
                if(bubble == this.bubbles.bRed && this.generateDirectionChange()) {
                    bubble.bottomWay = !bubble.bottomWay
                }
            }
            bubble.style.left = x + 'px'
            bubble.style.top  = y + 'px'
        },
        /***************************************************************************************************************
         * checkCollisions(): VALIDATES IF THE redBubble HITS ANY BLUE BUBBLE IN THE bubbles.list
         *      FOR THAT, CALLS THE areHitBoxesColliding(bubble) METHOD WICH RETURNS true IF THAT SO
         *      > ON HIT: GROWS THE redBubble IMAGE, REMOVES THE BLUE BUBBLE FROM THE bubbles.list,
         *                CLEARS THE INTERVAL THAT KEEPED MOVING THE BUBBLE AND REMOVES ITS ID FROM bubbles.intervalIDs
         *                UPDATES THE SCORE
         ***************************************************************************************************************/
        checkCollisions: function() {
            if(this.bubbles.list.length == 0)
                return
                
            for(let i = 0; i < this.bubbles.list.length; i++) {
                if(this.areHitBoxesColliding(this.bubbles.list[i])) {
                    this.bubbles.audio.pops[Math.floor(Math.random() * this.bubbles.audio.pops.length)].play()
                    this.bubbles.bRed.style.width  = (parseInt(this.bubbles.bRed.style.width)  + this.bubbles.growthRate) + 'px'
                    this.bubbles.bRed.style.height = (parseInt(this.bubbles.bRed.style.height) + this.bubbles.growthRate) + 'px'
                    this.bubbles.list.splice(i, 1)
                    clearInterval(this.bubbles.intervalIDs[i])
                    this.bubbles.intervalIDs.splice(i, 1)
                    this.game.totalPoints    += this.bubbles.extraPoints
                    this.bubbles.extraPoints  = (this.bubbles.list.length == 0)? 1 : (this.bubbles.extraPoints+1)
                    if(this.game.totalPoints % 10 == 0)
                        this.bubbles.step += 0.5

                    if((this.bubbles.const.initialWidth - this.bubbles.shrinkRate) > 10)
                        this.bubbles.shrinkRate += 0.5

                    return
                }
            }
        },
        /******************************************************************************************************
         * areHitBoxesColliding(bubble): RETURNS true IF THE bubble's HIT BOX HITS THE bubbles.bRed HIT BOX
         *      A HIT BOX LOOKS LIKE THIS:
         *      x1, y1 ------------- x2, y1
         *        |                     |
         *        |       BUBBLE        |
         *        |       IMAGE         |
         *        |                     |
         *      x1, y2 ------------- x2, y2
         * 
         ******************************************************************************************************/
        areHitBoxesColliding: function(bubble) {
            let redHB  = this.getHitBox(this.bubbles.bRed)
            let blueHB = this.getHitBox(bubble)
            // COLLISION CASE 1
            if(redHB.x1 >= blueHB.x1 && redHB.x1 <= blueHB.x2) {
                if(redHB.y1 >= blueHB.y1 && redHB.y1 <= blueHB.y2) {
                    return true // COLLISION CASE 1.1
                }
                if(redHB.y1 <= blueHB.y1 && redHB.y2 >= blueHB.y2) {
                    return true // COLLISION CASE 1.2
                }
                if(redHB.y2 >= blueHB.y1 && redHB.y2 <= blueHB.y2) {
                    return true // COLLISION CASE 1.3
                }
            }
            // COLLISION CASE 2
            if(redHB.x1 <= blueHB.x1 && redHB.x2 >= blueHB.x2) {
                if(redHB.y1 >= blueHB.y1 && redHB.y1 <= blueHB.y2) {
                    return true // COLLISION CASE 2.1
                }
                if(redHB.y2 >= blueHB.y1 && redHB.y2 <= blueHB.y2) {
                    return true // COLLISION CASE 2.2
                }
            }
            // COLLISION CASE 3
            if(redHB.x2 >= blueHB.x1 && redHB.x2 <= blueHB.x2) {
                if(redHB.y1 >= blueHB.y1 && redHB.y1 <= blueHB.y2) {
                    return true // COLLISION CASE 3.1
                }
                if(redHB.y1 <= blueHB.y1 && redHB.y2 >= blueHB.y2) {
                    return true // COLLISION CASE 3.2
                }
                if(redHB.y2 >= blueHB.y1 && redHB.y2 <= blueHB.y2) {
                    return true // COLLISION CASE 3.3
                }
            }
            return false
        },
        /***************************************************************************
         * getHitBox(bubble): CALCULATES AND RETURNS AN OBJECT { x1, x2, y1, y2 } 
         *      WITH THE COORDS OF THE bubble's HIT BOX 
         *      BASED ON THE POSITION AND SIZE OF THE bubble
         *      
         *      A HIT BOX LOOKS LIKE THIS:
         *      x1, y1 ------------- x2, y1
         *        |                     |
         *        |       BUBBLE        |
         *        |       IMAGE         |
         *        |                     |
         *      x1, y2 ------------- x2, y2
         * 
         ****************************************************************************/
        getHitBox(bubble) {
            let x1 = parseInt(bubble.style.left)
            let x2 = x1 + parseInt(bubble.style.width)
            let y1 = parseInt(bubble.style.top)
            let y2 = y1 + parseInt(bubble.style.height)
            return { x1, x2, y1, y2 }
        },
        /***********************************************************************************
         * generateRandomDirection(): GENERATES A RANDOM boolean DEPENDING ON THE CONDITION
         *  THIS IS USED ON createNewBubble() METHOD TO SET THE INITIAL DIRECTION
         *  ON THE newBubble
         ***********************************************************************************/
        generateRandomDirection: function() {
            return Math.random() > 0.5
        },
        /***********************************************************************************
         * generateDirectionChange(): GENERATES A RANDOM boolean DEPENDING ON THE CONDITION
         *      THIS METHOD IS USED ON validateCoords() 
         *      FOR THE RED BUBBLE'S MOVEMENT ALGORITHM EXCLUSIVELY
         *      HAS A 0,0007% CHANCE TO RETURN true. IN THIS CASE, THE RED BUBBLE
         *      WILL CHANGE DIRECTION SUDDENLY
         ***********************************************************************************/
        generateDirectionChange: function() {
            return Math.random() > 0.9993
        }
    }
}
</script>
<!--*************************************************************************************************************

     CSS/STYLES AREA

*************************************************************************************************************-->
<style scoped>
    .container {
        display: flex;
        justify-content: space-around;
        font-family: Verdana, Geneva, Tahoma, sans-serif;
        font-weight: bolder;
    }
    .totalPoints {
        color: aqua;
        font-size: 50px;
        background-color: rgba(0, 100, 255, 0.5);
        border-radius: 20px;
        width: 300px;
        text-align: center;
    }
    .extraPoints {
        color: aqua;
        font-size: 15px;
        background-color: rgba(0, 0, 200, 0.5);
        border-radius: 20px;
        text-align: center;
        font-weight: normal;
        padding: 5px;
    }    
    .availableBubbles {
        color: green;
        font-size: 20px;
        border-radius: 50px;
        background-color: rgba(0, 255, 100, 0.5);
        width: 300px;
        text-align: center;
    }
</style>