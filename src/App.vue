<template>
	<div id="app">
		<div class="header">
			<p>剩余：{{mine-flag}}</p>
			<button @click="restart(false)">当前难度：{{level+1}}</button>
			<p>用时：{{time}}</p>
		</div>
		<div class="center" :style="centerStyle">
			<div
				class="box"
				v-for="(item,index) in boxs"
				:key="index"
				:style="boxStyle"
				:class="handleBoxClass(item)"
				v-tap="e=>handleTapBox(index)"
				v-longtap="e=>handleLongBox(index)"
				v-swipeup="e=>handleLongBox(index)"
				v-swipedown="e=>handleLongBox(index)"
				v-swipeleft="e=>handleLongBox(index)"
				v-swiperight="e=>handleLongBox(index)"
			>
				<p v-if="item.value===-1 && over">{{item.status===2?'¤':'⊙'}}</p>
				<p v-else-if="item.status === 1">{{item.value}}</p>
				<p v-else-if="item.status === 2">※</p>
			</div>
		</div>
		<div class="bottom">
			<h4>难度选择</h4>
			<p
				v-for="(item,index) in opts"
				:key="index"
				@click="level=index;restart(false)"
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
			life: 0,
			over: false,
			init: false,
			clearId: null,
			opts: [
				{ row: 20, col: 15, mine: 50, scope: localStorage.scope1 || 0 },
				{
					row: 25,
					col: 18,
					mine: 100,
					scope: localStorage.scope2 || 0
				},
				{ row: 30, col: 20, mine: 200, scope: localStorage.scope3 || 0 }
			]
		};
	},
	mounted() {
		const _this = this;
		window.addEventListener("beforeunload", e => {
			let tips = "确定要关闭本游戏吗？";
			e.returnValue = tips;
			return tips;
		});
		window.addEventListener("error", e => {
			alert(`出错啦：${e.message}`);
		});
		this.restart();
		let key = window.prompt(
			"轻触=翻开格子\n按住格子往四周拽=插小旗子",
			"请输入金手指哦"
		);
		if (
			[
				"小怂怂",
				"小瑶瑶",
				"陈瑶",
				"陈优秀",
				"优秀",
				"我爱土鸡",
				"土鸡我爱你",
				"我喜欢你哦",
				"你好帅",
				"你好厉害"
			].indexOf(key) !== -1
		) {
			this.life = 3;
			this.handleSpeek("小怂怂我喜欢你哦");
		} else {
			this.life = 0;
			this.handleSpeek("你是谁？快夸我！");
		}
	},
	methods: {
		restart(auto) {
			auto = auto !== false;
			if (auto || window.confirm("确定要重新开始吗？")) {
				clearInterval(this.clearId);
				const level = this.opts[this.level];
				this.init = false;
				this.over = false;
				this.row = level.row;
				this.col = level.col;
				this.size = Math.floor(window.screen.availWidth / this.col);
				this.width = this.size * this.col;
				this.height = this.size * this.row;
				this.mine = level.mine;
				this.time = 0;
				this.flag = 0;
				this.boxs = [];
				for (let i = 0; i < this.row * this.col; i++) {
					this.boxs.push(new Box(0, 0));
				}
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
			if (this.over) {
				return false;
			}
			auto = auto === true;
			let box = this.boxs[index];
			if (!this.init) {
				this.handleInit(index);
			}
			if (box.status === 0) {
				if (box.value === 0) {
					box.trigger();
					let near = this.near(index);
					near.forEach(item => {
						this.handleTapBox(item, true);
					});
				}
				if (box.value === -1) {
					if (this.life > 0) {
						box.flag();
						this.life--;
						this.flag++;
						this.handleSpeek(
							`你还有${this.life}次机会踩到地雷自动插旗哦`
						);
					} else {
						this.over = true;
						this.handleSpeek("嘤嘤嘤，小怂怂你踩地雷了哦");
					}
				} else {
					box.trigger();
					this.handleover();
				}
			}
		},
		handleSpeek(str, alert) {
			alert = alert !== false;
			try {
				let ssu = new SpeechSynthesisUtterance(str);
				speechSynthesis.speak(ssu);
			} catch (e) {
				window.alert(e.message);
			}
			alert && window.alert(str);
		},
		handleLongBox(index) {
			if (this.over) {
				return false;
			}
			let box = this.boxs[index];
			box.flag();
			if (box.status === 2) {
				this.flag++;
			} else {
				this.flag--;
			}
			this.handleover();
		},
		handleover() {
			this.over = this.boxs.every(item => item.status !== 0);
			if (this.over) {
				clearInterval(this.clearId);
				if (this.time < this.opts[this.level].scope) {
					this.opts[this.level].scope = localStorage[
						"scope" + (this.level + 1)
					] = this.time;
					this.handleSpeek(
						`小怂怂越来越聪明了哦！这次居然只用了${
							this.time
						}秒就赢了哦`
					);
				} else {
					this.handleSpeek("小怂怂真厉害呀又赢了哦");
				}
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
				margin: 2% auto;
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
					&.flag {
						background-color: darkgreen;
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
