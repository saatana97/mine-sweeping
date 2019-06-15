<template>
	<div id="app">
		<div class="header">
			<p>{{mine-flag}}</p>
			<button @click="restart">当前难度：{{level+1}}</button>
			<p>{{time}}</p>
		</div>
		<div class="center" :style="centerStyle">
			<div
				class="box"
				v-for="(item,index) in boxs"
				:key="index"
				:style="boxStyle"
				:class="handleBoxClass(item)"
				v-tap="e=>handleTapBox(index)"
				v-longtap="e=>handleLongBox(item,index)"
				v-swipeup="e=>handleLongBox(item,index)"
				v-swipedown="e=>handleLongBox(item,index)"
				v-swipeleft="e=>handleLongBox(item,index)"
				v-swiperight="e=>handleLongBox(item,index)"
			>
				<p v-if="item.status === 1">{{item.value}}</p>
				<p v-if="item.status === 2">⊙</p>
			</div>
		</div>
		<div class="bottom">
			<h4>难度选择</h4>
			<p
				v-for="(item,index) in opts"
				:key="index"
				@click="level=index;restart()"
			>{{`${item.row}行${item.col}列${item.mine}个地雷，最佳成绩${item.scope}秒`}}</p>
			<h6>❤❤❤最爱小怂怂的土鸡</h6>
		</div>
	</div>
</template>

<script>
export default {
	name: "App",
	data() {
		return {
			size: 0,
			width: 0,
			height: 0,
			row: 0,
			col: 0,
			mine: 0,
			flag: 0,
			time: 0,
			level: 0,
			boxs: [],
			win: false,
			init: false,
			clearId: null,
			opts: [
				{ row: 20, col: 15, mine: 30, scope: localStorage.scope1 || 0 },
				{ row: 30, col: 20, mine: 60, scope: localStorage.scope2 || 0 },
				{ row: 40, col: 30, mine: 100, scope: localStorage.scope3 || 0 }
			]
		};
	},
	mounted() {
		this.restart();
		alert("轻触=翻开格子\n按住格子往四周拽=插小旗子");
	},
	methods: {
		restart() {
			clearInterval(this.clearId);
			const level = this.opts[this.level];
			this.init = false;
			this.win = false;
			this.row = level.row;
			this.col = level.col;
			this.size = Math.floor(window.screen.availWidth / this.col);
			this.width = this.size * 10;
			this.height = this.width * 2;
			this.mine = level.mine;
			this.time = 0;
			this.flag = 0;
			this.boxs = [];
			for (let i = 0; i < this.row * this.col; i++) {
				this.boxs.push(new Box(0, 0));
			}
		},
		random(min, max) {
			return Math.round(Math.random() * (max - min) + min);
		},
		near(index) {
			let near = [],
				top = false,
				right = false,
				bottom = false,
				left = false;
			if (index >= this.col) {
				top = true;
				near.push(index - this.col);
			}
			if ((index + 1) % this.col !== 0) {
				right = true;
				near.push(index + 1);
			}
			if (index < (this.row - 1) * this.col) {
				bottom = true;
				near.push(index + this.col);
			}
			if ((index + 1) % this.col !== 1) {
				left = true;
				near.push(index - 1);
			}
			if (top && left) {
				near.push(index - this.col - 1);
			}
			if (top && right) {
				near.push(index - this.col + 1);
			}
			if (bottom && right) {
				near.push(index + this.col + 1);
			}
			if (bottom && left) {
				near.push(index + this.col - 1);
			}
			return near;
		},
		handleInit(ignore) {
			this.init = true;
			const _this = this;
			let count = this.mine;
			let mines = [];
			do {
				let index = this.random(0, this.boxs.length - 1);
				if (index !== ignore && mines.indexOf(index) === -1) {
					this.boxs[index].value = -1;
					let near = this.near(index);
					near.forEach(item => {
						let box = _this.boxs[item];
						if (box.value !== -1) {
							box.value++;
						}
					});
					mines.push(index);
					count--;
				}
			} while (count > 0);
			this.clearId = setInterval(() => {
				this.time++;
			}, 1000);
		},
		handleTapBox(index, auto) {
			auto = auto === true;
			let box = this.boxs[index];
			if (!this.init) {
				this.handleInit(index);
			}
			if (box.status === 0) {
				box.trigger();
				if (box.value === 0) {
					let near = this.near(index);
					console.info(index, near);
					near.forEach(item => {
						this.handleTapBox(item, true);
					});
				} else if (box.value === -1) {
					alert("小怂怂你踩地雷了哦");
					this.restart();
				}
			}
		},
		handleLongBox(item) {
			item.flag();
			if (item.status === 2) {
				this.flag++;
			} else {
				this.flag--;
			}
		},
		handleWin() {
			this.win = this.boxs.every(item => item.status !== 0);
			if (this.win) {
				clearInterval(this.clearId);
				this.opts[this.level].scope = localStorage[
					"scope" + (this.level + 1)
				] = this.time;
			}
		},
		handleBoxClass(item) {
			let res = { flag: item.status === 2, active: item.status === 1 };
			res["level" + item.value] = true;
			return res;
		}
	},
	computed: {
		boxStyle() {
			return {
				width: this.size + "px",
				height: this.size + "px",
				lineHeight: this.size + "px",
				fontSize: this.size * 0.6 + "px"
			};
		},
		centerStyle() {
			return {
				width: this.size * this.col + "px",
				height: this.size * this.row + "px"
			};
		}
	}
};
class Box {
	/**
	 * value:-1是地雷，自然数为周围地雷的数量
	 * status:0为未知，1为已知，2为插旗
	 */
	constructor(value, status) {
		this.value = value;
		this.status = status;
	}
	clone() {
		return new Box(this.value, this.status);
	}
	trigger() {
		if (this.status === 0) {
			this.status = 1;
		}
	}
	flag() {
		if (this.status === 0) {
			this.status = 2;
		} else if (this.status === 2) {
			this.status = 0;
		}
	}
}
</script>
<style lang="scss">
html {
	width: 100%;
	height: 100%;
	* {
		margin: 0;
		padding: 0;
		position: relative;
	}
	body {
		width: 100%;
		height: 100%;
		#app {
			width: 100%;
			height: 100%;
			user-select: none;
			background-color: black;
			color: white;
			overflow: auto;
			.header {
				display: flex;
				justify-content: space-around;
			}
			.center {
				display: flex;
				flex-wrap: wrap;
				.box {
					box-sizing: border-box;
					border: 1px outset #ccc;
					background-color: green;
					text-align: center;
					&.active {
						background-color: gray;
						border-style: groove;
					}
					&.flag {
						background-color: darkgreen;
					}
					&.level0 {
						color: gray;
					}
					&.level1 {
						color: white;
					}
					&.level2 {
						color: khaki;
					}
					&.level3 {
						color: lightsalmon;
					}
					&.level4 {
						color: peru;
					}
					&.level5 {
						color: orange;
					}
					&.level6 {
						color: skyblue;
					}
					&.level7 {
						color: purple;
					}
					&.level8 {
						color: red;
					}
				}
			}
			.bottom {
				h4 {
					text-align: center;
				}
				p {
					border: 1px skyblue outset;
					padding: 2% 5%;
					margin: 2% 10%;
					width: 70%;
				}
				h6 {
					text-align: right;
					color: indianred;
				}
			}
		}
	}
}
</style>
