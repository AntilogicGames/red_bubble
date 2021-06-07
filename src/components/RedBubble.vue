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
                growthRate      : 2,          // RED BUBBLE's GROWTH RATE USED EVERYTIME A BLUE ONE IS 
                shrinkRate      : 0,          // BLUE BUBBLE's SHRINK RATE USED EVERYTIME A BLUE ONE IS 
                step            : 1,
                isFirstBubble   : true,
                maxBlueBubbles  : 5,
                extraPoints     : 1,
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
                    initialWidth   : 50,
                    initialHeight  : 50,
                    margin         : 30
                }
            }
        }
    },
    created: function() {
        window.onclick = this.createNewBubble
        setInterval(() => {
                this.checkCollisions()
        }, 1)
    },
    methods: {
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
                rightWay   : this.generateBool(),
                bottomWay  : this.generateBool(),
                rightStep  : Math.floor(Math.random() * this.bubbles.step) + 1,
                bottomStep : Math.floor(Math.random() * this.bubbles.step) + 1,
                rotateDeg  : 0
            }
            this.bubbles.currentID++
            this.bubbles.extraPoints     = 1      // RESET extraPoints EVERYTIME THE PLAYER CLICKS
            this.moveBubble(newBubble)
        },
        generateInitialCoords: function(mouseEvent) {
            let deltaX = Math.floor(Math.random() * 30) + 100
            let deltaY = Math.floor(Math.random() * 30) + 100
            let x      = (mouseEvent.clientX < this.game.windowWidth /2)? (mouseEvent.clientX + deltaX) : (mouseEvent.clientX - deltaX)
            let y      = (mouseEvent.clientY < this.game.windowHeight/2)? (mouseEvent.clientY + deltaY) : (mouseEvent.clientY - deltaY)
            return { x, y }
        },
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
                if(bubble == this.bubbles.bRed && this.generateBoolLowProb()) {
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
                if(bubble == this.bubbles.bRed && this.generateBoolLowProb()) {
                    bubble.bottomWay = !bubble.bottomWay
                }
            }
            bubble.style.left = x + 'px'
            bubble.style.top  = y + 'px'
        },
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
        getHitBox(bubble) {
            let x1 = parseInt(bubble.style.left)
            let x2 = x1 + parseInt(bubble.style.width)
            let y1 = parseInt(bubble.style.top)
            let y2 = y1 + parseInt(bubble.style.height)
            return { x1, x2, y1, y2 }
        },
        generateBool: function() {
            return Math.random() > 0.5
        },
        generateBoolLowProb: function() {
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