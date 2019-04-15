<template>
	<view class="container">
        <view class="article" v-for="(article,index) in articles" :key="index">
			<!-- 标题 -->
			<text class="article-title" @tap="gotoDetail(article.id)">{{article.title}}</text>
			<br />
			<!-- 大于等于三张图片的显示方式 -->
			<view class="" v-if="article.imgs.length >= 3">
				<view class="thumbnail-box">
					<view class="thumbnail-item" v-for="(item,index1) in article.imgs" :key="index1" v-if="index1<3">
						<image :src="item.imgUrl"></image>
					</view>
				</view>
			</view>
			<!-- 小于三张图片的显示方式 -->
			<view class="" v-else-if="article.imgs.length >= 1">
				<view class="text-img-box">
					<view class="left">
						<text>{{handleContent(article.content)}}...</text>
					</view>
					<view class="right">
						<image :src="article.imgs[article.imgs.length - 1].imgUrl"></image>
					</view>
				</view>
			</view>
			<!-- 没有图片的显示方式 -->
			<view class="text-box" v-else>
				<text>{{article.title}}</text>
			</view>
			<!-- 文章作者等信息 -->
			<view class="article-info">
				
				<image :src="article.avatar" class="avatar small"></image>
				<view class="right2">
				<text class="info-text">{{article.nickname}}</text>
				<br />
				<text class="info-text">{{handleTime(article.createTime)}}</text>
				
				</view>
			</view>
			<view class="jg"></view>
		</view>
		<button class="circle-btn" @tap="gotoWrite"><text class="icon-text">+</text></button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				articles:[]
			}
		},
		onLoad:function() {
			this.getArticles();
		},
		onShow: function() {
			var _this = this;
			const loginKey = uni.getStorageSync('login_key');
			if (loginKey) {
				//console.log(loginKey);
				this.storageData = {
					login: loginKey.login,
					login: loginKey.nickname
				}
			} else {
				this.storageData = {
					login: false
				};
			}
			uni.request({
				url: 'http://192.168.43.26:8080/api/user/' + uni.getStorageSync('login_key').userId,
				method: 'GET',
				header: { 'content-type': 'application/json' },
				success: res => {
					if (res.data.code === 0) {
						console.log(res.data.data.avatar+'————————————');
						_this.avatar = res.data.data.avatar;
						_this.nickname = res.data.data.nickname;
					}
				}
			});
		},
		onPullDownRefresh:function(){
			this.getArticles();
		},
		methods: {
			getArticles:function(){
				var _this = this;
				uni.request({
					url: this.apiServer + '/article/list',
					method:'GET',
					header:{'content-type': 'application/x-www-form-urlencdded'},
					success: res => {
						_this.articles = res.data.data;
					},
					complete:function(){
						uni.stopPullDownRefresh();
					}
				});
			},
			gotoDetail:function(aId){
				uni.navigateTo({
					url:'../article_detail/article_detail?aId=' +aId
				});
			},
			gotoWrite:function(){
				if(uni.getStorageSync('login_key').login === true){
				uni.navigateTo({
					url:'../write/write'
				});
				}else{
					uni.navigateTo({
						url:'../signin/signin'
					});
				}
			},
			handleTime: function(date) {
				var d = new Date(date);
				var year = d.getFullYear();
				var month = d.getMonth() + 1;
				var day = d.getDate() < 10 ? '0' + d.getDate() : '' + d.getDate();
				var hour = d.getHours() < 10 ? '0' + d.getHours() : '' + d.getHours();
				var minutes = d.getMinutes() < 10 ? '0' + d.getMinutes() : '' + d.getMinutes();
				var seconds = d.getSeconds() < 10 ? '0' + d.getSeconds() : '' + d.getSeconds();
				return year + '-' + month + '-' + day + ' ' + hour + ':' + minutes + ':' + seconds;
			},
			handleContent:function(content){
				content = content.replace(/(\n)/g,'');
				content = content.replace(/(\t)/g,'');
				content = content.replace(/(\r)/g,'');
				content = content.replace(/<\/?[^>]*>/g,'');
				content = content.replace(/\s*/g,'');
				return content.substring(0,50);
			}
		}
	};
</script>

<style>
	.article-box{
		height: auto;
		width: 100%;
	}
	.article{
		display: flex;
		flex-direction: column;
	}
	.article-info{
		color: darkgray;
	}
	.article-title{
		font-weight:bold;
	}
	.thumbnail-box{
		margin-top: 10px;
		display: flex;
		height: 120px;
		width: 100%;
		/* border-bottom: 1px solid #E6E7E9; */
	}
	.thumbnail-item{
		flex: 1 1 33%;
		height: 110px;
		margin-right: 5px;
		margin-left: 5px;
	}
	.thumbnail-item image{
		width: 100%;
		height: 100%;
	}
	.avatar{
		width: 50px;
		height: 50px;
		margin-top: 25px;
	}
	.info-text{
		margin-left: 10px;
		margin-top: 10px;
		color: black;
	}
	.text-img-box{
		display: flex;
	}
	.left{
		flex: 1 1 70%;
	}
	.right{
		flex: 1 1 30%;
	}
	.right image{
		width: 100%;
		height: 100%;
	}
	.right2{
		margin-left: 50px;
		margin-top: -55px;
	}
	.jg{
	 width: 100%;
	 height: 9px;
	 background-color:#EEEEEE;
	 margin-top: 10px;
 }
	.icon-text {
	color: #fff;
	}
	.circle-btn {
	position: fixed;
	bottom: 60px;
	right: 10px;
	width: 45px;
	height: 45px;
	border-radius: 50%;
	background-color: #de533a;
	background: linear-gradient(40deg, #ffd86f, #fc6262);
	/* background-image: url(../../static/Pencil32.png); */
	box-shadow: 2px 5px 10px #aaa;
	cursor: pointer;
	border: none;
	outline: none;
	display: flex;
	justify-content: center;
	align-items: center;
}
</style>
