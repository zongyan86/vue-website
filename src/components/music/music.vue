<template>
  <div class="music">
  	<div class="music_bg" v-if="getIsAPP.isHigher768" :style="{background:'url(' + getCurrentMusic.picurl + ') center center / cover'}">
  		<div class="mask_bg"></div>
  	</div>
  	<div class="mask_linear_bg" v-if="getCurrentMusic.picurl && getIsAPP.isHigher768"></div>
  	<div class="music_content">
  		<div class="music_body">
  			<div class="left_list">
  				<div class="music_home">
  					<div v-if="getIsAPP.isHigher768" class="select_button">
  						<router-link tag="span" to="/music/playlist" class="todo_btn playing_btn">
				        	正在播放
				        </router-link>
			  			<router-link tag="span" to="/music/collection" class="todo_btn collect_btn">
				        	我的收藏
				        </router-link>
				        <router-link tag="span" to="/music/toplist" class="todo_btn top_btn">
				        	排行榜/歌单
				        </router-link>
			  			<router-link tag="span" to="/music/search" class="todo_btn search_btn">
				        	搜索音乐
				        </router-link>
			  		</div>
			  		<div v-if="!getIsAPP.isHigher768" class="select_m_button">
			  			<router-link tag="span" to="/music/playlist" class="todo_btn playing_btn">
				        	正在播放
				        </router-link>
			  			<router-link tag="span" to="/music/collection" class="todo_btn collect_btn">
				        	收藏
				        </router-link>
				        <router-link tag="span" to="/music/toplist" class="todo_btn top_btn">
				        	排行榜/歌单
				        </router-link>
			  			<router-link tag="span" to="/music/search" class="todo_btn search_btn">
				        	搜索
				        </router-link>
			  		</div>
			  		<transition name="silde-top">
						<router-view class="list_content" name="listinfo"></router-view>
					</transition>
  				</div>
  			</div>
  			<transition name="silde-top">
				<router-view class="music_wrapper" name="fullscreen"></router-view>
			</transition>
  			<div class="right_info">
  				<div class="bg-info">
  					<img class="music-bg" :src="getCurrentMusic.picurl ? getCurrentMusic.picurl : 'http://www.daiwei.org/vue/bg/18627925998890855.jpg'">
  				</div>
  				<div class="lrc-content" ref="lrcContent">
  					<div class="lrc-wrapper" ref="lrcWrapper">
  						<p class="lrc-item" v-if="!getMusicLrcLists" >纯音乐,请欣赏</p>
  						<p class="lrc-item" v-if="getMusicLrcLists" v-for="(item, key, index) in getMusicLrcLists" :class="getCurrentMusicLrcIndex === index + 1 ? `active` : ``">{{item}}</p>
  					</div>
  				</div>
  			</div>
  		</div>
  		<div class="music_ctrl">
  			<div class="left_ctrl">
  				<div class="music_detail_ctrl">
  					<i class="playPrev icon-prevdetail" @click.stop="playPrev()" title="上一曲 ctrl + left"></i>
  					<i class="playPause" :class="getAudioIsPlay ? 'icon-pause' : 'icon-play'" @click.stop="playPause()" title="播放/暂停 ctrl + space"></i>
  					<i class="playNext icon-nextdetail" title="下一曲 ctrl + right" @click.stop="playNext()"></i>
  				</div>
  				<div class="music_progress" id="music_progress">
  					<div class="music_current_detail">
  						<span class="music_c_name">{{getCurrentMusic.name ? getCurrentMusic.name : '未曾遗忘的青春'}} - {{getCurrentMusic.singer ? getCurrentMusic.singer : '戴维戴维'}}</span>
  						<span class="music_c_time">{{getMusicCurrentT !== NaN & getMusicCurrentT !== '00:00' ? getMusicDurationType(getMusicCurrentT * 1000) : '00:00'}} / {{getCurrentMusic.duration ? getCurrentMusic.duration : '00:00'}}</span>
  					</div>
  					<div class="music_progress_bar" id="music_progress">
  						<div class="duration" id="music_progressD" @click="clickProgress">
  							<div class="buffering" :style="{width:`${bufferingP}%`}"></div>
  							<div class="real" :style="{width: getMusicPro}"></div>
  						</div>
  						<div class="range" @mousedown="dragMouseDown" @touchstart="dragTouchStart" @touchmove="dragTouchMove" @touchend="dragTouchEnd" :style="{left:`${getMusicPro}`}"></div>
  					</div>
  				</div>
  			</div>
  			<div class="right_ctrl">
  				
  			</div>
  		</div>
  	</div>
  </div>
</template>
<script>
  // import fecth from './../../utils/fecth.js'
  import store from '../../store'
  import musicApi from './music.js'
  // import axios from 'axios'
  // import qs from 'qs'
  export default {
  	data () {
  		return {
  			isDrag: false,
  			currentMusic: {},
  			bufferingP: 0,
  			top_list_all: {
				0: ['云音乐新歌榜', '/discover/toplist?id=3779629'],
				1: ['云音乐热歌榜', '/discover/toplist?id=3778678'],
				2: ['网易原创歌曲榜', '/discover/toplist?id=2884035'],
				3: ['云音乐飙升榜', '/discover/toplist?id=19723756'],
				4: ['云音乐电音榜', '/discover/toplist?id=10520166'],
				5: ['UK排行榜周榜', '/discover/toplist?id=180106'],
				6: ['美国Billboard周榜', '/discover/toplist?id=60198'],
				7: ['KTV嗨榜', '/discover/toplist?id=21845217'],
				8: ['iTunes榜', '/discover/toplist?id=11641012'],
				9: ['Hit FM Top榜', '/discover/toplist?id=120001'],
				10: ['日本Oricon周榜', '/discover/toplist?id=60131'],
				11: ['韩国Melon排行榜周榜', '/discover/toplist?id=3733003'],
				12: ['韩国Mnet排行榜周榜', '/discover/toplist?id=60255'],
				13: ['韩国Melon原声周榜', '/discover/toplist?id=46772709'],
				14: ['中国TOP排行榜(港台榜)', '/discover/toplist?id=112504'],
				15: ['中国TOP排行榜(内地榜)', '/discover/toplist?id=64016'],
				16: ['香港电台中文歌曲龙虎榜', '/discover/toplist?id=10169002'],
				17: ['华语金曲榜', '/discover/toplist?id=4395559'],
				18: ['中国嘻哈榜', '/discover/toplist?id=1899724'],
				19: ['法国 NRJ EuroHot 30周榜', '/discover/toplist?id=27135204'],
				20: ['台湾Hito排行榜', '/discover/toplist?id=112463'],
				21: ['Beatport全球电子舞曲榜', '/discover/toplist?id=3812895']
			}
  		}
  	},
  	methods: {
  		searchMusic (e) {
  			musicApi.searchMusic(e.target.innerHTML, 1, this)
  		},
  		// 专辑信息
  		getAlbum (id) {
  			musicApi.getMusicAlbum(id, this)
  		},

  		// 获取歌词
  		getMusicLrc (id) {
  			musicApi.getMusicLrc(id, this)
  		},

  		// 音乐时长格式
  		getMusicDurationType (time) {
  			return musicApi.getMusicDurantionType(time, this)
  		},
  		// 初始化音乐播放器
		initMusic () {
			musicApi.getMusicSheet(124995419, this)
  		},
  		AudiEle () {
  			return store.getters.getAudioEle
  		},
  		playPrev () {
  			musicApi.playNextPrev(this, false)
  		},
  		playNext () {
  			musicApi.playNextPrev(this, true)
  		},
  		playPause () {
  			musicApi.playPause()
  		},
  		keypress () {
  			let that = this
  			document.onkeydown = e => {
				if (e && e.ctrlKey && e.keyCode === 32) {   // 同时按下 ctrl + 空格
					musicApi.playPause()
				}
				if (e && e.ctrlKey && e.keyCode === 39) { 	// 同时按下 ctrl + -->
					musicApi.playNextPrev(that, true)
				}
				if (e && e.ctrlKey && e.keyCode === 37) { 	// 同时按下 ctrl + <--
					musicApi.playNextPrev(that, false)
				}
  			}
  		},
  		initAudioEvent () {
  			const audio = store.getters.getAudioEle
  			musicApi.initAudioEvent(this, audio)
  		},
  		// 鼠标点击拖动事件
  		dragMouseDown (e) {
  			musicApi.dragMouseDown(this, e)
  		},
  		// 点击跳转位置
  		clickProgress (e) {
  			musicApi.clickProgress(e)
  		},
  		// dragMouseMove (e) {
  		// 	musicApi.dragMouseMove(this, e)
  		// },
  		// dragMouseUp (e) {
  		// 	musicApi.dragMouseUp(this, e)
  		// },
  		// dragMouseLeave (e) {
  		// 	musicApi.dragMouseLeave(this, e)
  		// }
  		dragTouchStart (e) {
  			musicApi.dragTouchStart(this, e)
  		},
  		dragTouchMove (e) {
  			musicApi.dragTouchMove(this, e)
  		},
  		dragTouchEnd (e) {
  			e.stopPropagation()
			e.preventDefault()
  			musicApi.dragTouchEnd(this, e)
  		}
  	},
  	computed: {
  		musicList () {
  			return store.getters.getMusicList
  		},
  		getCurrentMusic () {
  			return store.getters.getCurrentAudio
  		},
  		getMusicLrcLists () {
  			return store.getters.getCurrentAudio.lyric
  		},
  		getCurrentMusicLrcIndex () {
			return store.getters.getAudiolrcIndex
  		},
  		getIsAPP () {
  			return store.getters.getGlobalInfo
  		},
  		getAudioIsPlay () {
  			return store.getters.getAudioIsPlay
  		},
  		getMusicPlayingList () {
  			return store.getters.getMusicPlayList
  		},
  		getMusicCurrentT () {
			return store.getters.getAudioCurrentTime
  		},
  		getMusicPro () {
  			const mp = store.getters.getAudioCurrentD
  			return (mp).toFixed(4) + '%'
  		}
  	},
  	watch: {
  		// currentMusic (newval, oldval) {
  		// 	alert(newval.lyric)
  		// },
  		'$route' (to, from) {
			store.dispatch({
				type: 'set_MusicRouter',
				data: to.path
			})
        }
  	},
  	mounted () {
  		store.commit({
  			type: 'setAudioLrcContent',
  			data: this.$refs.lrcContent
  		})
  		// this.searchMusic()
  		// this.initAudioEvent()
  		// this.keypress()
  		// musicApi.musicEvent(this)
  		this.$nextTick(() => {
  			this.initAudioEvent()
  			this.keypress()
  		})
  	}
  }
</script>
<style lang="stylus" rel="stylesheet/stylus">
	@import '../../common/stylus/global.styl'
	@import '../../common/stylus/border-1px/index.styl'
	.music
		position:fixed
		top:0
		left:0;
		bottom:0
		width:100%
		right:0
		border-radius:12px
		.music_bg
			position:absolute
			top:0
			left:0;
			bottom:0
			width:100%
			right:0
			z-index:-1
			backdrop-filter: blur(10px)
			filter: blur(16px)
			background-color:#000
			transition:all 0.8s
			.mask_bg
				position:absolute
				top:0
				left:0;
				bottom:0
				width:100%
				right:0
				background:rgba(0,0,0,0.5)
		.mask_linear_bg
			position:absolute
			top:0
			left:0;
			bottom:0
			width:100%
			right:0
			z-index:-2
			background:#000
		.music_content
			position:fixed
			top:80px
			bottom:30px
			left:50%
			right:100px
			left:100px
			// max-width:1240px
			// width:100%
			// transform:translate3d(-50%,0,0)
			margin:0 auto
			overflow-y:scroll
			box-sizing:border-box
			-webkit-overflow-scrolling: touch
			padding:15px;
			box-sizing:border-box
			// background:rgba(0,0,0,0.5)
			overflow:hidden
			.music_body
				width:100%
				height: calc(100% - 80px)
				// background:red
				// font-size:0
				display:flex
				position: relative
				.left_list
					width:100%
					height:100%
					overflow: hidden
					position: relative
					.music_home
						// display:inline-block
						flex: 1 1 auto
						width:100%
						height:100%
						// background:#A99E9E
						vertical-align:top
						.select_m_button
							width:100%
							height:60px
							display:flex
							align-items:center
							.todo_btn
								width:auto
								height:24px
								line-height:24px
								color:$text_before_color
								border:1px solid $border_bottom_color_deep
								border-radius:2px
								padding:0px 7px
								margin:10px
								font-size:12px
								cursor:pointer
								&:hover,&.active
									color:$text_before_color
									border:1px solid $text_before_color				
								
						.select_button
							width:100%
							height:60px
							display:flex
							align-items:center
							.todo_btn
								width:auto
								height:32px
								line-height:32px
								color:$text_before_color
								border:1px solid $border_bottom_color_deep
								border-radius:2px
								padding:0 15px
								margin:10px
								font-size:14px
								cursor:pointer
								&:hover,&.active,&.router-link-active
									color:$text_before_color
									border:1px solid $text_before_color						
						.list_content
							height:calc(100% - 60px)
							padding:10px
							box-sizing:border-box
							overflow:auto
							.music_list_title,.music_list
								height:50px
								line-height:50px
								// background:red
								font-size:0
								span
									display:inline-block
									font-size:14px
									color:$text_before_color
									text-overflow:ellipsis
									overflow:hidden
									white-space:nowrap
									vertical-align:middle
									&.music_name
										width:calc(50% - 50px)
										font-size:14px
									&.music_singer
										width:20%
										padding:0 5px
										box-sizing:border-box
									&.music_zhuanji
										width:20%
										padding:0 5px
										box-sizing:border-box
									&.music_duration
										width:10%
										padding:0 5px
										box-sizing:border-box
									&.music_index
										width:50px
										height:100%
										text-align: center;
									span
										cursor:pointer
										&:hover
											text-decoration:underline
								.music_name
									width:calc(50% - 50px)
									display:inline-block
									font-size:14px
									height:50px
									line-height:50px
									color:$text_before_color
									text-overflow:ellipsis
									overflow:hidden
									white-space:nowrap
									font-size:0
									position:relative
									vertical-align: middle;
									.span_name
										display:inline-block
										width: 100% 
										height:100%
										// background:red
										font-size:14px
									.hover_menu
										position:absolute
										width:60px
										height:100%
										right:0
										top:0
										// background:red
										display:none
										i
											font-size:18px
											display:inline-block
											width:36px
											height:36px
											line-height:36px
											color:$border_bottom_color_deep
											text-align:center
											border-radius:50%
											vertical-align:middle
											border:2px solid $border_bottom_color_deep
											cursor:pointer
											&:hover
												color:$text_before_color
												border:2px solid $text_before_color
										
								&.border-1px
									border-1px($border_bottom_color,bottom)
							.music_list_content
								height:calc(100% - 50px)
								position:relative
								overflow:auto
								.music_list
									position:relative
									&:hover
										.music_name
											.hover_menu
												display:block
				.right_info
					// display:inline-block
					flex: 0 0 310px
					width:310px
					max-width:100%
					height:100%
					// background:#6ED367
					vertical-align:top
					position:relative
					.bg-info
						position: relative
						display: block
						width: 186px
						height: 186px					
						margin: 0 auto
						// display: flex;
						// align-items: center;
						&::after
							content:''
							background:url('./../../../static/cd_block.png') 0 0 no-repeat
							position:absolute
							top:0
							left:0
							right: -20px
							bottom:0
						.music-bg
							vertical-align: middle;
							width: 176px
							height: 176px
							border:none
							position:absolute
							top:5px
							left:5px
							border-radius: 4px
					.lrc-content
						position:absolute
						top:200px
						left:0
						right:0
						bottom:15px
						overflow:hidden
						padding:20px 0
						box-sizing:border-box
						.lrc-item
							width:100%
							height:auto
							min-height:26px
							line-height:26px
							text-align:center
							margin:0
							color:$text_before_color
							font-size:12px
							&.active
								color:#A7EEBE
			.music_ctrl
				width:100%;
				height:80px
				// background:rgba(0,0,0,0.3)
				display:flex
				.left_ctrl
					flex: 1 1 auto
					display:flex
					.music_detail_ctrl
						display:flex
						align-items:center
						i
							color:$text_before_color
							font-size:38px
							margin:0 10px
							transition:0.5s
							cursor:pointer
							&.playPause
								font-size:46px
							&:hover
								color:$text_color
					.music_progress
						width: 100%
						padding: 0 20px
						box-sizing:border-box
						display:flex
						align-items:center
						flex-direction:column
						justify-content: center
						.music_current_detail
							display:block
							width:100%
							height:auto
							color:$text_before_color
							margin-bottom: 10px
							font-size:0
							overflow:hidden
							.music_c_name
								display:inline-block
								// text-align:left
								width:calc(100% - 115px)
								white-space:nowrap
								overflow:hidden
								text-overflow:ellipsis
								font-size:16px
								vertical-align:top
								// float:left
							.music_c_time
								display:inline-block
								font-size:16px
								vertical-align:top
								text-align:right
								white-space:nowrap
								overflow:hidden
								text-overflow:ellipsis
								// text-align:right
								width:110px
								// float:right
									
						.music_progress_bar
							width:100%
							height:2px
							// background:$text_color_opacity
							box-sizing:border-box
							position:relative
							cursor:pointer
							&:before{
								content: ''
								position:absolute
								bottom:2px
								width:100%
								height:10px
								background:transparent
							}
							.duration
								width:100%
								height:2px
								position:relative
								background:$progress_color
								border-radius: 1px
								.buffering
									width:20%
									height:100%
									background:$buffering_color
									border-radius: 1px
									position:absolute
									top:0
									left:0
									transition:width 0.3s
								.real
									width:10%
									position:absolute
									height:100%
									left:0
									background:$real_color
									border-radius: 1px
									// transition:width 0.3s
							.range
								// position:absolute
								// top:-4px
								// right:-4px
								// width:10px
								// height:10px
								// border-radius: 50%
								// background:$range_color
								// cursor:pointer
								width: 6px
								height: 6px
								margin-top: -3px
								margin-left: 0
								border-radius: 50%
								background-color: #f00
								position: absolute
								left: 0
								top: 50%
								z-index: 2
								cursor:pointer
								&:before
									content: " ";
									display: block;
									position: absolute;
									width: 24px;
									height: 24px;
									border-radius: 50%;
									background: radial-gradient(rgba(0, 0, 0, 0) 20%, rgba(255, 255, 255, 0.6) 50%, rgba(0, 0, 0, 0) 60%);
									top: 50%;
									margin-top: -12px;
									margin-left: -9px;
									cursor: pointer;
									outline: 0;
									-webkit-tap-highlight-color: transparent;
								
								
				.right_ctrl
					flex: 0 0 310px
					// background:blue
				@media screen and (max-width: 922px) 
					.right_ctrl
						display:none
		@media screen and (max-width: 1440px) 
			.music_content
				top:40px
				bottom:20px
				right:20px
				left:20px
		@media screen and (max-width: 1200px)
			.music_content
				.music_body
					.left_list
						.list_content
							.music_list_content
								.music_list
									position:relative
									&:hover
										.music_name
											.hover_menu
												display:none
		@media screen and (max-width: 998px)
			.music_content
				.music_body
					.left_list
						.list_content
							.music_list
								.music_name
									width:calc(60% - 50px)
								.music_zhuanji
									display:none!important
								.music_singer
									width:30%!important
								.music_duration
									width:20%!important
							.music_list_title
								span
									&.music_name
										font-size:14px
										width:calc(60% - 50px)
									&.music_zhuanji
										display:none
									&.music_singer
										width:30%!important
									&.music_duration
										width:20%!important								
		@media screen and (max-width: 922px)
			.music_content
				.music_body
					.left_list
						
						// .list_content
						// 	.music_list
						// 		.music_name
						// 			width:calc(60% - 50px)
						// 		.music_zhuanji
						// 			display:none
						// 		.music_singer
						// 			width:30%!important
						// 	.music_list_title
						// 		span
						// 			&.music_name
						// 				font-size:14px
						// 				width:calc(60% - 50px)
						// 			&.music_zhuanji
						// 				display:none
						// 			&.music_singer
						// 				width:30%										
					.right_info
						position:absolute
						top:0
						left:0
						right:0
						bottom:0
						width:100%
						background:red
						display:none
		@media screen and (max-width: 525px)
			.music_content
				top: 40px
				bottom: 0px
				right: 0px
				left: 0px
				padding:0
				.music_body
					height:calc(100% - 110px)
					.left_list
						.select_button
							.todo_btn
								padding:0 10px
								overflow:hidden
								text-overflow:ellipsis
								overflow:hidden
								font-size:14px
								height:32px
								line-height:32px
						.list_content
							.music_list
								.music_duration
									// display:none!important
									width:20%!important
								.music_singer
									width:30%!important
								.music_name
									width:calc(100% - 50px)				
							.music_list_title
								display:none
						.music_list_content
							height:100%!important
				.music_ctrl
					height:110px
					.left_ctrl
						flex-direction: column
						.music_detail_ctrl
							justify-content:space-between
							padding:0 50px
							margin-bottom:6px
							i
								font-size:32px!important
							.playPause 
								font-size:38px!important
						.music_progress
							.music_current_detail
								margin-bottom:3px
								position:relative
								
	video 
		display:none	
</style>
