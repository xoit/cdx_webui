<!-- Right side first row， Toolbar -->
<template>
	<div class="tools-panel">
		<div class="tools-left">
			<span title="折叠菜单" class="tool-fox" v-if="$root.isOpen == true" @click="$root.endOpen()">
				<i class="el-icon-s-fold"></i>
			</span>
			<span title="展开菜单" class="tool-fox" v-if="$root.isOpen == false" @click="$root.startOpen()">
				<i class="el-icon-s-unfold"></i>
			</span>
			<span title="搜索-input" class="tool-fox search-fox" :class=" isSearch ? 'search-fox-show' : '' ">
				<el-select v-model="searchText" size="mini" filterable placeholder="请输入菜单关键字" ref="search" 
					@change="findMenuBySearch" @blur="closeSearch" @keyup.esc.native="closeSearch">
					<el-option v-for="item in searchList" :key="item.id" :label="item.text" :value="item.id"></el-option>
				</el-select>
			</span>
			<span title="搜索菜单" class="tool-fox" @click="closeSearch()" v-if="!isShowSearchInput">
				<i class="el-icon-search" style="font-weight: bold;"></i>
			</span>
			<span title="搜索菜单" class="tool-fox" @click="startSearch()" v-else>
				<i class="el-icon-search" style="font-weight: bold;"></i>
			</span>
			<span title="刷新" class="tool-fox" @click="$root.f5Tab($root.nativeTab)">
				<i class="el-icon-refresh-right" style="font-weight: bold;"></i>
			</span>
			<span title="当前时间" class="tool-fox">
				<span style="font-size: 0.90em;">{{nowTime}}</span>
			</span>
		</div>
		<div class="tools-right">
			<span title="点击登录" class="tool-fox" onclick="location.href='login.html'" v-if="$root.user == null">
				<span style="font-size: 0.8em; font-weight: bold; position: relative; top: -2px;">未登录</span>
			</span>
			<span title="我的信息" class="tool-fox user-info" style="padding: 0;" v-if="$root.user != null">
				<el-dropdown @command="handleCommand" trigger="click" size="medium">
					<span class="el-dropdown-link user-name" style="height: 100%; padding: 0 1em; display: inline-block;">
						<img :src="$root.user.avatar" class="user-avatar">
						<span>{{$root.user.username}}</span>
						<i class="el-icon-arrow-down el-icon--right"></i>
					</span>
					<el-dropdown-menu slot="dropdown">
						<el-dropdown-item v-for="drop in $root.dropList" :command="drop.name" :key="drop.name">{{drop.name}}</el-dropdown-item>
					</el-dropdown-menu>
				</el-dropdown>
			</span>
			<span title="Themes" class="tool-fox" style="padding: 0;">
				<el-dropdown @command="toggleTheme" trigger="click" size="medium">
					<span class="el-dropdown-link" style="height: 100%; padding: 0 1em; display: inline-block;">
						<i class="el-icon-price-tag" style="font-weight: bold;"></i>
						<span style="font-size: 0.9em;">Themes</span>
					</span>
					<el-dropdown-menu slot="dropdown">
						<el-dropdown-item :command="t.value" v-for="t in themeList" :key="t.name">
							<span :style=" $root.themeV == t.value ? 'color: #44f' : '' ">{{t.name}} </span>
						</el-dropdown-item>
					</el-dropdown-menu>
				</el-dropdown>
			</span> 
			<!-- Remove stick note
			<span title="便签" class="tool-fox" @click="openNote()">
				<i class="el-icon-edit" style="font-weight: bold; font-size: 0.9em;"></i>
				<span style="font-size: 0.9em;">便签</span>
			</span>
			-->
			<span title="Full Screen" class="tool-fox" v-if="isFullScreen == false" @click="fullScreen()">
				<i class="el-icon-rank" style="font-weight: bold; transform: rotate(45deg)"></i>
			</span>
			<span title="Exit Full Screen" class="tool-fox" v-if="isFullScreen == true" @click="outFullScreen()">
				<i class="el-icon-bottom-left" style="font-weight: bold; "></i>
			</span>
		</div>
		<!-- tab被拖拽时的遮罩（tab上拖拽：新窗口打开） -->
		<div class="shade-fox" v-if="$root.isDrag" 
			@dragover="$event.preventDefault();" 
			@drop="$event.preventDefault(); $event.stopPropagation(); $root.newWinTab($root.dragTab);">
			<span style="font-size: 16px;">新窗口打开</span>
		</div>
	</div>
</template>

<script>
	module.exports = {
		data() {
			return {
				isSearch: false,	// 当前是否处于搜索模式 
				isShowSearchInput: true,	// 是否显示打开搜索图标 
				searchText: '',		// 搜索框已经输入的字符 
				searchList: [],			// 搜索框 待选列表 
				
				isFullScreen: false,	// 是否处于全屏状态 
				
				nowTime: 'Loading...'	,	// 当前时间 
				currInterval: null,		// 刷新当前时间的定时器 
				
				themeList: [	// 主题数组
					{name: 'Blue', value: '1'},
					{name: 'Green', value: '2'},
					{name: 'White', value: '3'},
					{name: 'Grey', value: '4'},
					{name: 'Red', value: '5'},
					{name: 'Purple', value: '9'},
					{name: 'Pro Golden', value: '6'},
					{name: 'Dark Blue', value: '7'},
					{name: 'Plain Grey', value: '8'},
					{name: 'Grass Green', value: '10'},
				],
				
			}
		},
		methods: {
			// ------------------------------ 搜索相关 ------------------------------
			// 开启搜索
			startSearch: function() {
				this.searchText = '';
				this.isSearch = true;
				this.f5SearchList();
				setTimeout(function() {
					this.isShowSearchInput = false;
					this.$refs['search'].focus();	//.$refs['nav-tool-bar'].
				}.bind(this), 200);
			},
			// 关闭搜索
			closeSearch: function() {
				this.searchText = '';
				this.isSearch = false;
				setTimeout(function() {
					try{
						this.isShowSearchInput = true;
						document.querySelector('body>.el-select-dropdown.el-popper').style.display = 'none';
					}catch(e){throw e}
				}.bind(this), 200);
			},
			// 查找菜单 
			findMenuBySearch: function(id) {
				this.$root.showMenuById(id);
				this.closeSearch();
			},
			// 刷新待选列表 
			f5SearchList: function() {
				var searchList = [];
				
				let index = 1;
				function push(id, str) {
					searchList.push({id: id, text: (index++) + ". " + str});
				}
				
				// 遍历菜单 
				let childList = this.$root.menuList;
				let showList = this.$root.showList;
				for (let menu1 of childList) {
					if(menu1.isShow === false || showList.indexOf(menu1.id + '') == -1) continue;
					if(menu1.childList) {
						for (let menu2 of menu1.childList) {
							if(menu2.isShow === false || showList.indexOf(menu2.id + '') == -1) continue;
							if(menu2.childList) {
								for (let menu3 of menu2.childList) {
									if(menu3.isShow === false || showList.indexOf(menu3.id + '') == -1) continue;
									if(menu3.childList) {
										for (let menu4 of menu3.childList) {
											if(menu4.isShow === false || showList.indexOf(menu4.id + '') == -1) continue;
											push(menu4.id, menu1.name + ' > ' + menu2.name + ' > ' + menu3.name + ' > ' + menu4.name);
										}
									} else {
										push(menu3.id, menu1.name + ' > ' + menu2.name + ' > ' + menu3.name);
									}
								}
							} else {
								push(menu2.id, menu1.name + ' > ' + menu2.name);
							}
						}
					} else {
						push(menu1.id, menu1.name);
					}
				}
				
				this.searchList = searchList;
			},
			
			// ------------------------------ Themes ------------------------------
			// Switch theme
			toggleTheme: function(command) {
				// 开始切换
				this.$root.themeV = command + "";
				localStorage.setItem('themeV', command);
				for (var i = 0; i < this.themeList.length; i++) {
					if(this.themeList[i].value + '' == command + '') {
						this.$message('Switch to ' + this.themeList[i].name);
					}
				}
			},
			
			// ------------------------------ 全屏 ------------------------------
			// 进入全屏 
			fullScreen: function() {
				this.isFullScreen = true;
				if(document.documentElement.RequestFullScreen){
					document.documentElement.RequestFullScreen();
				}
				//兼容火狐
				if(document.documentElement.mozRequestFullScreen){
					document.documentElement.mozRequestFullScreen();
				}
				//兼容谷歌等可以webkitRequestFullScreen也可以webkitRequestFullscreen
				if(document.documentElement.webkitRequestFullScreen){
					document.documentElement.webkitRequestFullScreen();
				}
				//兼容IE,只能写msRequestFullscreen
				if(document.documentElement.msRequestFullscreen){
					document.documentElement.msRequestFullscreen();
				}
			},
			// 退出全屏
			outFullScreen: function() {
				this.isFullScreen = false;
				if(document.exitFullScreen){
					document.exitFullscreen()
				}
				//兼容火狐
				if(document.mozCancelFullScreen){
					document.mozCancelFullScreen()
				}
				//兼容谷歌等
				if(document.webkitExitFullscreen){
					document.webkitExitFullscreen()
				}
				//兼容IE
				if(document.msExitFullscreen){
					document.msExitFullscreen()
				}
			},
			
			// ------------------------------ 其它 ------------------------------
			// 处理userinfo的下拉点击
			handleCommand: function(command) {
				this.$root.dropList.forEach(function(drop) {
					if(drop.name == command) {
						drop.click();
					}
				})
			},
			// 打开便签
			openNote: function() {
				var w = (document.body.clientWidth * 0.4) + 'px';
				var h = (document.body.clientHeight * 0.6) + 'px';
				var default_content = '一个简单的小便签, 关闭浏览器后再次打开仍然可以加载到上一次的记录, 你可以用它来记录一些临时资料';
				var value = localStorage.getItem('sa_admin_note') || default_content;
				var index = layer.prompt({
					title: '一个小便签', 
					value: value,
					formType: 2,
					area: [w, h],
					btn: ['保存'],
					maxlength: 99999999,
					skin: 'layer-note-class' 
				}, function(pass, index){
					layer.close(index)					
				});
				var se = '#layui-layer' + index + ' .layui-layer-input';
				var d = document.querySelector(se);
				d.oninput = function() {
					localStorage.setItem('sa_admin_note', this.value);
				}
			},
			
			// 刷新时间
			initInterval: function() {
				if(this.currInterval) {
					clearInterval(this.currInterval);
				}
				// Update time
				this.currInterval = setInterval(function() {
					var da = new Date();
					var Y = da.getFullYear(); // Year
					var M = da.getMonth() + 1; // Month
					var D = da.getDate(); // Day
					var h = da.getHours(); // Hour
					var sx = "am";
					if (h >= 6) {
						sx = "am"
					}
					if (h >= 12) {
						sx = "pm";
						if (h >= 18) {
							sx = "pm";
						}
						h -= 12;
					}
					var m = da.getMinutes(); // Minute
					var s = da.getSeconds(); // Second
					var z = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'][da.getDay()] ; //周几
					// z = z == 0 ? 'Sun' : z;
					var zong = "";
				
					//zong += Y + "-" + M + "-" + D + " " + sx + " " + h + ":" + m + ":" + s + " 周" + z;
					zong += Y + "-" + M + "-" + D + " " + " " + h + ":" + m + ":" + s + " " + sx + ", "+ z;
					this.nowTime = zong;
				}.bind(this), 1000);
			}
		
		},
		created() {
			this.initInterval();
		}
	}
</script>

<style scoped>
	
	.tools-left{border: 0px #000 solid; float: left;}
	.tools-right{float: right;}
	.tool-fox{padding: 0 1em; display: inline-block; cursor: pointer;}
	.tool-fox, .tool-fox i{transition: all 0.2s;}
	
	.user-info{position: relative; top: -2px;}
	.user-avatar{width: 30px; height: 30px; border-radius: 50%; vertical-align: middle;}
	.user-info .user-name{font-size: 0.9em;} 
	
	/* 搜素框 */
	.search-fox{display: inline-block; vertical-align: middle; overflow: hidden; max-width: 0px; padding: 0em 0em; margin-left: -5px; transition: all 0.2s;}
	.search-fox-show{display: inline-block; max-width: 500px; margin-left: 0px; padding: 0 1em;}
	.search-fox:hover{background-color: rgba(0,0,0,0) !important;}
	.search-fox .el-input__inner{border-radius: 0px; border-width: 0px; border-bottom-width: 1px; background-color: rgba(0,0,0,0);}
	.search-fox .el-input__icon{display: none;}
	
	/*800之下*/
	@media(max-width: 800px) {
		.tools-right{display: none;}
	}
</style>
