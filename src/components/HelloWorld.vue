<template>
  <div class="hello">
    <button @click="makeMap()">Make a map</button>
    <div class="map" v-if="mapData" tabindex="0" @keydown.up="up"
     @keydown.down="down"
     @keydown.left="left"
     @keydown.right="right">
      <!-- Iterate rows -->
      <div>
        xLength: {{ xLength }}<br/>
        yLength: {{ yHeight }}<br/>
        xStart: {{ xStart }}<br/>
        yStart: {{ yStart }}<br/>
      </div>
      <div v-html="mapHtml"></div>
    </div>
  </div>
</template>

<script>
import * as _ from "lodash";
export default {
  name: 'HelloWorld',
  data() {
    return {
      xLength: 50,
      yHeight: 50,
      xStart: 0,
      yStart: 0,
      mapData: null,
      mapHtml: "",
      chanceToBuild: 0.5,
      spaces: ["X", "s", " ", "t", "e"],
      playerCoords: null,
      previousSpace: null

    }
  },
  computed: {
    
  },
  methods: {
    up(event) {
      if(this.playerCoords.y > 0 && this.mapData[this.playerCoords.y -1][this.playerCoords.x] !== "X") {
        this.movePlayer({ x: this.playerCoords.x, y: this.playerCoords.y - 1});
      }
      console.log(event);
    },
    down(event) {
      if(this.playerCoords.y < this.yHeight && this.mapData[this.playerCoords.y + 1][this.playerCoords.x] !== "X") {
        this.movePlayer({ x: this.playerCoords.x, y: this.playerCoords.y + 1});
      }
      console.log(event);
    },
    left(event) {
      if(this.playerCoords.x > 0 && this.mapData[this.playerCoords.y][this.playerCoords.x - 1] !== "X") {
        this.movePlayer({ x: this.playerCoords.x - 1, y: this.playerCoords.y});
      }
      console.log(event);
    },
    right(event) {
      if(this.playerCoords.x < this.xLength - 1 && this.mapData[this.playerCoords.y][this.playerCoords.x + 1] !== "X") {
        this.movePlayer({ x: this.playerCoords.x + 1, y: this.playerCoords.y});
      }
      console.log(event);
    },
    movePlayer(coords) {
      this.mapData[this.playerCoords.y][this.playerCoords.x] = this.playerCoords.previousValue;
      this.playerCoords.y = coords.y;
      this.playerCoords.x = coords.x;
      this.playerCoords.previousValue = this.mapData[this.playerCoords.y][this.playerCoords.x];
      this.mapData[this.playerCoords.y][this.playerCoords.x] = "<span style='background-color:green;color:white'>@</span>";
      this.drawMap();
    },
    shiftArray(array) {
      var first = array[0];
      var newArray = array.splice(1,array.length);
      newArray.push(first);
      return newArray;
    },
    makeMap() {
      /*
      Legend:
      X = wall,
      S = start,
      T = treasure,
      E = enemy,
      blank = empty space
      */

      // default map, all closed spaces
      var array = Array.apply(null, Array(this.yHeight)).map( () => { 
        return Array.apply(null, Array(this.xLength)).map(() => { return "X"; })
      });

      // get random starting point, just not 0,0 because it breaks
      this.xStart = this.randomIntFromInterval(1, this.xLength - 1);
      this.yStart = this.randomIntFromInterval(1, this.yHeight - 1);
      //array[this.yStart][this.xStart] = "<span style='color:red;'>S</span>";

      // initiate map
      this.playerCoords = { x: this.xStart, y: this.yStart, previousValue: "<span style='color:red;'>S</span>" };
      array[this.yStart][this.xStart] = "<span style='background-color:green;color:white'>@</span>";

      // set coordinates to starting point, follow path generator
      var xCurr = this.xStart;
      var yCurr = this.yStart;
      var usedSpaces = 0;
      var xNew, yNew;

      // track which direction we are moving, and try that direction first, i.e. if we hit left, then the order is left, up, right down
      // if we hit up, then the order is up, right, down, left
      var directions = ["left","up","right","down"];
      var currDirection = "left";

      while(usedSpaces <= (this.xLength * this.yHeight)/3) {
        console.log("at " + xCurr + ", " + yCurr);
        var built = false;
        _.forEach(directions, direction => {
          if(!built) {
            switch(direction) {
              case "left":
                // check left for OOB
                if(xCurr != 0) {
                  xNew = xCurr - 1;
                  yNew = yCurr;

                  // is it possible to build here
                  if(array[yNew][xNew] === "X") {
                    // should we build here?
                    if(Math.random() < this.chanceToBuild) {
                      // what to build? 
                      //var item = this.randomIntFromInterval(2, this.spaces.length);
                      array[yNew][xNew] = "."; // this.spaces[item];
                      // move to new coord and break
                      xCurr = xNew;
                      yCurr = yNew;
                      usedSpaces++;
                      currDirection = "left";
                      built = true;
                      console.log("built left");
                    }
                  }
                }
                break;
              case "up":
                // check top for OOB
                if(yCurr != 0) {
                  xNew = xCurr;
                  yNew = yCurr - 1;

                  // is it possible to build here
                  if(array[yNew][xNew] === "X") {
                    // should we build here?
                    if(Math.random() < this.chanceToBuild) {
                      // what to build? 
                      //var item = this.randomIntFromInterval(2, this.spaces.length);
                      array[yNew][xNew] = "."; // this.spaces[item];
                      // move to new coord and break
                      xCurr = xNew;
                      yCurr = yNew;
                      usedSpaces++;
                      currDirection = "up";
                      built = true;
                      console.log("built up");
                    }
                  }
                }
                break;
              case "right":
                // check right for OOB
                if(xCurr != this.xLength - 1) {
                  xNew = xCurr + 1;
                  yNew = yCurr;

                  // is it possible to build here
                  if(array[yNew][xNew] === "X") {
                    // should we build here?
                    if(Math.random() < this.chanceToBuild) {
                      // what to build? 
                      //var item = this.randomIntFromInterval(2, this.spaces.length);
                      array[yNew][xNew] = "."; // this.spaces[item];
                      // move to new coord and break
                      xCurr = xNew;
                      yCurr = yNew;
                      usedSpaces++;
                      currDirection = "right";
                      built = true;
                      console.log("built right");
                    }
                  }
                }
                break;
              case "down":
                // check bottom for OOB
                if(yCurr != this.yHeight - 1) {
                  xNew = xCurr;
                  yNew = yCurr + 1;

                  // is it possible to build here
                  if(array[yNew][xNew] === "X") {
                    // should we build here?
                    if(Math.random() < this.chanceToBuild) {
                      // what to build? 
                      //var item = this.randomIntFromInterval(2, this.spaces.length);
                      array[yNew][xNew] = "."; // this.spaces[item];
                      // move to new coord and break
                      xCurr = xNew;
                      yCurr = yNew;
                      usedSpaces++;
                      currDirection = "down";
                      built = true;
                      console.log("built down");
                    }
                  }
                }
                break;
            }
          }
          
        });
      
        if(built) {
          while(currDirection !== directions[0]) {
            directions = this.shiftArray(directions);
          }
          continue;
        } else {
          // nothing built, move to first empty space or try again
          // note: at 0,0 gets stuck in a loop moving right and left, just move back to start for now and try it all again
          // if(xCurr === 0 && yCurr === 0) {
          //   xCurr = this.xStart;
          //   yCurr = this.yStart;
          //   console.log("back to start");
          //   continue;
          // }
          var moved = false;
          _.forEach(directions, direction => {
            if(!moved) {
              switch(direction) {
                case "left":
                  if(xCurr != 0 && array[yCurr][xCurr -1] !== "X") {
                    xCurr = xCurr - 1;
                    console.log("moved left");
                    moved = true;
                    currDirection = "left";
                  }
                  break;
                case "up": 
                  if (yCurr != 0 && array[yCurr - 1][xCurr] !== "X") {
                    yCurr = yCurr - 1;
                    console.log("moved up");
                    moved = true;
                    currDirection = "up";
                  }
                  break;
                case "right":
                  if (xCurr != this.xLength - 1 && array[yCurr][xCurr + 1] !== "X") {
                    xCurr = xCurr + 1;
                    console.log("moved right");
                    moved = true;
                    currDirection = "right";
                  }
                  break;
                case "down":
                  if (yCurr != this.yHeight - 1 && array[yCurr + 1][xCurr] !== "X") {
                    yCurr = yCurr + 1;
                    console.log("moved down");
                    moved = true;
                    currDirection = "down";
                  } 
                  break;
              }
            }
              
          });
          if(moved) {
            while(currDirection !== directions[0]) {
              directions = this.shiftArray(directions);
            }
          }
          else {
            // couldn't move, what do?
            // shuffle directions
            directions = this.shiftArray(directions);
            console.log("trying again");
            continue;
          }
        }
      }
      console.log("finished");
      this.mapData = array;
      this.drawMap();
    },
    drawMap() {
      var mapString = ""; // "x: " + this.playerCoords.x + " y: " + this.playerCoords.y;
      // iterate rows
      _.forEach(this.mapData, (row) => {
        //mapString += "Row " + (i < 10 ? "0" + i : i) + " ";
        _.forEach(row, col => {
          mapString+=col;
        });
        mapString+="<br/>";
      });
      
      this.mapHtml = mapString;
    },
    randomIntFromInterval(min, max) {
      // min and max included
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.map {
    font-family: "Courier New", Courier, monospace;
    line-height: 10px;
    letter-spacing: 1px;
    color: #999;
    cursor: default;
    border: 1px solid black;
}
</style>
