<template>
	<div class="home">
		<div class="info">
			<div class="title">2048</div>
			<div class="scores">
				<div class="score">
					{{score}}
					<div v-if="scoreAddition" class="score-addition">+{{scoreAddition}}</div>
				</div>
				<div class="best">{{best}}</div>
			</div>
			<div class="clear-both"></div>
			<div class="play-again" @click="playAgain">PLAY AGAIN</div>
		</div>

		<The2048Game ref="game" @addScore="addScore" @unsolvable="unsolvable"/>
	</div>
</template>

<script>
  // @ is an alias to /src
  import The2048Game from '../components/2048Game.vue'

  export default {
    name: 'Home',
    components: {
      The2048Game
    },
    data() {
      return {
        score: 0,
        best: 0,
        scoreAddition: 0,
      };
    },
    methods: {
      addScore( num ) {
        this.scoreAddition = num;
        this.score += num;
        if ( this.score > this.best ) this.best = this.score;
        setTimeout( () => {
          this.scoreAddition = 0;
        }, 1000 )
      },
      unsolvable() {
        alert( 'There is no answer! Please play again!' );
      },
      playAgain() {
        this.score = 0;
        this.$refs.game.playAgain();
      },
    },
  }
</script>

<style lang="scss" scoped>
	.home {
		font-size: 40px;

		$color: #bbada0;

		.info {
			margin: 50px auto;
			width: 620px;

			&:after {
				display: block;
				content: " ";
				clear: both;
			}

			.title {
				float: left;
				margin-top: 0px;
				font-size: 80px;
				line-height: 130px;
				font-weight: bold;
				color: #8f7a66;
			}

			.scores {
				float: right;
				font-size: 50px;
				font-weight: bold;
				color: #fff;

				.score, .best {
					position: relative;
					display: inline-block;
					margin: 0 10px;
					padding: 30px 20px;
					height: 60px;
					line-height: 100px;
					background: $color;
					border-radius: 6px;
					text-align: center;
					min-width: 80px;

					&:before {
						content: "SCORE";
						position: absolute;
						left: 0;
						top: 20px;
						width: 100%;
						font-size: 26px;
						line-height: 1;
						text-align: center;
						color: #eee4da;
					}

					.score-addition {
						position: absolute;
						right: 10px;
						font-size: 50px;
						animation: score-addition 1s ease-in-out both;
						@keyframes score-addition {
							0% {
								top: 20px;
								opacity: 1;
							}
							100% {
								top: -100px;
								opacity: 0;
							}
						}
					}
				}

				.best:before {
					content: "BEST";
				}
			}

			.clear-both {
				clear: both;
			}

			.play-again {
				margin: 20px 10px;
				float: right;
				display: inline-block;
				padding: 20px 20px;
				background: #8f7a66;
				border-radius: 6px;
				text-decoration: none;
				color: #f9f6f2;
				height: 40px;
				line-height: 42px;
				text-align: center;
			}
		}
	}
</style>