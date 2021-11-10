<template>
	<div class="trigle-crush-game"
	     @touchstart="onTouchStart"
	     @mousedown="onMouseDown"
	     @touchmove="onTouchMove"
	     @mousemove="onMouseMove">
		<div class="grid">
			<template v-for="(_cells,x) in cells">
				<template v-for="(cell,y) in _cells">
					<div class="cell" :x="x" :y="y"></div>
				</template>
			</template>
		</div>
		<div class="tiles">
			<template v-for="tile in tiles">
				<div class="tile" :key="tile.id"
				     :class="[tile.status]"
				     :x="tile.x" :y="tile.y" :tile="tile.num">
					{{tile.num}}
				</div>
			</template>
		</div>
	</div>
</template>

<script>
  const W = 4, H = 4;
  export default {
    name: 'The2048Game',
    props: {},
    data() {
      return {
        animating: false,
        cells: [],
        tiles: [],
        lastTileId: 0,
      };
    },
    mounted() {
      window.addEventListener( 'keydown', this.onKeyDown );

      this.init();
    },
    destroyed() {
      window.removeEventListener( 'keydown', this.onKeyDown );
    },
    methods: {
      // 初始化
      init() {
        let cells = [];
        for ( let x = 0; x < W; ++x ) {
          cells.push( [] );
          for ( let y = 0; y < H; ++y ) {
            cells[ x ].push( 0 );
          }
        }
        this.cells = cells;
        this.tiles = [];
        this.generateTile();
        this.generateTile();
      },
      // 重玩
      playAgain() {
        this.init();
      },
      // 生成新的棋子
      generateTile() {
        const emptyCells = this.__getEmptyCells();
        if ( !emptyCells.length ) return;
        let { x, y } = emptyCells[ Math.floor( Math.random() * emptyCells.length ) ];

        let tile = {
          id: ++this.lastTileId,
          x,
          y,
          num: 2,
        };
        this.tiles.push( tile );
        this.cells[ x ][ y ] = tile;
      },
      // 寻找空格
      __getEmptyCells() {
        let emptyCells = [];
        for ( let x = 0; x < W; ++x ) {
          for ( let y = 0; y < H; ++y ) {
            if ( !this.cells[ x ][ y ] ) emptyCells.push( { x, y } );
          }
        }
        return emptyCells;
      },
      addScore( num ) {
        this.$emit( 'addScore', num );
      },
      gameOver() {
        this.$emit( 'gameOver' );
      },
      // 监听滑动事件
      onTouchStart( e ) {
        if ( this.animating ) return;
        const touch = e.touches[ 0 ];
        this._touchStartTileX = touch.clientX;
        this._touchStartTileY = touch.clientY;
      },
      onTouchMove( e ) {
        if ( this.animating ) return;
        if ( !this._touchStartTileX ) return;
        const touch = e.touches[ 0 ];
        this.__checkMoveTiles( touch );
      },
      onMouseDown( e ) {
        if ( this.animating ) return;
        this._touchStartTileX = e.clientX;
        this._touchStartTileY = e.clientY;
      },
      onMouseMove( e ) {
        if ( this.animating ) return;
        if ( !this._touchStartTileX ) return;
        this.__checkMoveTiles( e );
      },
      onKeyDown( e ) {
        switch ( e.code ) {
          case 'ArrowUp':
            this.__moveTiles( 'up' );
            break;
          case 'ArrowDown':
            this.__moveTiles( 'down' );
            break;
          case 'ArrowLeft':
            this.__moveTiles( 'left' );
            break;
          case 'ArrowRight':
            this.__moveTiles( 'right' );
            break;
        }
      },
      __checkMoveTiles( touch ) {
        const deltaX = touch.clientX - this._touchStartTileX;
        const deltaY = touch.clientY - this._touchStartTileY;
        const threshold = 30;
        if ( Math.abs( deltaX ) > threshold && Math.abs( deltaY ) < threshold / 2 ) {
          if ( deltaX < 0 ) {
            this.__moveTiles( 'left' );
          } else {
            this.__moveTiles( 'right' );
          }
          this._touchStartTileX = 0;
        } else if ( Math.abs( deltaX ) < threshold / 2 && Math.abs( deltaY ) > threshold ) {
          if ( deltaY < 0 ) {
            this.__moveTiles( 'up' );
          } else {
            this.__moveTiles( 'down' );
          }
          this._touchStartTileX = 0;
        } else if ( Math.abs( deltaX ) > threshold / 2 && Math.abs( deltaY ) > threshold / 2 ) {
          this._touchStartTileX = 0;
        }
      },
      // 棋子移动
      __moveTiles( direction ) {
        if ( this.animating ) return;
        // console.log( '__moveTiles', direction )
        let lastSingleTile, nextX = 0, nextY = 0;
        let canMove = false;
        switch ( direction ) {
          case 'up':
            for ( let x = 0; x < W; ++x ) {
              lastSingleTile = null;
              nextY = 0;
              for ( let y = nextY; y < H; ++y ) {
                const tile = this.cells[ x ][ y ];
                if ( !tile ) continue;
                if ( lastSingleTile && lastSingleTile.num === tile.num ) {
                  this.__concatTiles( tile, lastSingleTile );
                  lastSingleTile = null;
                  canMove = true;
                } else {
                  if ( y > nextY ) {
                    this.__moveTile( tile, x, nextY );
                    canMove = true;
                  }
                  lastSingleTile = tile;
                  ++nextY;
                }
              }
            }
            break;
          case 'down':
            for ( let x = 0; x < W; ++x ) {
              lastSingleTile = null;
              nextY = H - 1;
              for ( let y = nextY; y >= 0; --y ) {
                const tile = this.cells[ x ][ y ];
                if ( !tile ) continue;
                if ( lastSingleTile && lastSingleTile.num === tile.num ) {
                  this.__concatTiles( tile, lastSingleTile );
                  lastSingleTile = null;
                  canMove = true;
                } else {
                  if ( y < nextY ) {
                    this.__moveTile( tile, x, nextY );
                    canMove = true;
                  }
                  lastSingleTile = tile;
                  --nextY;
                }
              }
            }
            break;
          case 'left':
            for ( let y = 0; y < H; ++y ) {
              lastSingleTile = null;
              nextX = 0;
              for ( let x = nextX; x < W; ++x ) {
                const tile = this.cells[ x ][ y ];
                if ( !tile ) continue;
                if ( lastSingleTile && lastSingleTile.num === tile.num ) {
                  this.__concatTiles( tile, lastSingleTile );
                  lastSingleTile = null;
                  canMove = true;
                } else {
                  if ( x > nextX ) {
                    this.__moveTile( tile, nextX, y );
                    canMove = true;
                  }
                  lastSingleTile = tile;
                  ++nextX;
                }
              }
            }
            break;
          case 'right':
            for ( let y = 0; y < H; ++y ) {
              lastSingleTile = null;
              nextX = W - 1;
              for ( let x = nextX; x >= 0; --x ) {
                const tile = this.cells[ x ][ y ];
                if ( !tile ) continue;
                if ( lastSingleTile && lastSingleTile.num === tile.num ) {
                  this.__concatTiles( tile, lastSingleTile );
                  lastSingleTile = null;
                  canMove = true;
                } else {
                  if ( x < nextX ) {
                    this.__moveTile( tile, nextX, y );
                    canMove = true;
                  }
                  lastSingleTile = tile;
                  --nextX;
                }
              }
            }
            break;
        }

        if ( canMove ) {
          this.animating = true;

          setTimeout( () => {
            this.animating = false;
            this.generateTile();
            if ( this.__checkGameOver() ) {
              setTimeout( () => {
                this.gameOver();
              }, 200 );
            }
          }, 200 );
        }
      },
      __concatTiles( fromTile, toTile ) {
        const { x, y } = fromTile;
        fromTile.x = toTile.x;
        fromTile.y = toTile.y;
        fromTile.removed = true;
        this.cells[ x ][ y ] = 0;
        setTimeout( () => {
          toTile.num *= 2;
          this.addScore( toTile.num );
          this.tiles.splice( this.tiles.indexOf( fromTile ), 1 );
        }, 200 );
      },
      __moveTile( tile, x, y ) {
        // console.log( '__moveTile', tile.x, tile.y, x, y );
        this.cells[ tile.x ][ tile.y ] = 0;
        this.cells[ x ][ y ] = tile;
        tile.x = x;
        tile.y = y;
      },
      __removeTile( tile, x, y ) {
        this.cells[ x ][ y ] = 0;
        this.tiles.splice( this.tiles.indexOf( tile ), 1 );
      },
      // 计算是否结束
      __checkGameOver() {
        for ( let x = 0; x < W; ++x ) {
          for ( let y = 0; y < H; ++y ) {
            if ( !this.cells[ x ][ y ] ) return false;
            if ( x > 0 && this.cells[ x - 1 ][ y ] && this.cells[ x - 1 ][ y ].num === this.cells[ x ][ y ].num ) return false;
            if ( y > 0 && this.cells[ x ][ y - 1 ] && this.cells[ x ][ y - 1 ].num === this.cells[ x ][ y ].num ) return false;
          }
        }
        return true;
      },
    },
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
	$X: 4;
	$Y: 4;
	$W: 130px;
	$H: 130px;
	$gap: 20px;

	.trigle-crush-game {
		position: relative;
		margin: 0 auto;
		padding: 20px;
		width: ($W + $gap) * $X - $gap;
		height: ($H + $gap) * $Y - $gap;
		border-radius: 10px;
		background: #c0aba0;

		.grid, .tiles {
			position: absolute;
			left: 20px;
			top: 20px;
		}

		.grid .cell,
		.tiles .tile {
			position: absolute;
			width: $W;
			height: $H;
			border-radius: 5px;
			background: #d8cdc3;

			@for $x from 0 to $X {
				&[x="#{$x}"] {
					left: ($W+$gap) * $x;
				}
			}
			@for $y from 0 to $Y {
				&[y="#{$y}"] {
					top: ($H+$gap) * $y;
				}
			}
		}


		.tiles {
			.tile {
				position: absolute;
				width: $W;
				height: $H;
				color: #fff;
				font-size: 70px;
				font-weight: bold;
				line-height: $H;
				text-align: center;
				border-radius: 6px;
				transition: left 0.2s, top 0.2s ease;

				animation: tile-appear 0.2s ease 0.1s both;
				@keyframes tile-appear {
					0% {
						opacity: 0;
						transform: scale(0);
					}
					100% {
						opacity: 1;
						transform: scale(1);
					}
				}

				&[tile="2"] {
					color: #75695e;
					background: #eee4da;
				}

				&[tile="4"] {
					color: #75695e;
					background: #ede0c8;
				}

				&[tile="8"] {
					background: #f2b179;
				}

				&[tile="16"] {
					background: #f59563;
				}

				&[tile="32"] {
					background: #f67c5f;
				}

				&[tile="64"] {
					background: #f65e3b;
				}

				&[tile="128"] {
					font-size: 50px;
					background: #edcf72;
				}

				&[tile="256"] {
					font-size: 50px;
					background: #edcc61;
				}

				&[tile="512"] {
					font-size: 50px;
					background: #edc850;
				}

				&[tile="1024"] {
					font-size: 40px;
					background: #edc53f;
				}

				&[tile="2048"] {
					font-size: 40px;
					background: #edc22e;
				}
			}
		}
	}
</style>
