<template>
	<view class="container">
		<view class="detail-box">
			<text class="article-title">{{ article.title }}</text>
			<view class="article-info-box">
				<view class="article-info">
					<image :src="article.avatar" class="small-avatar"></image>
					<view class="right2">
						<text class="right-nickename">{{ article.nickname }}</text>
						<br />
						<text class="right-lou2">{{ article.createTime }}</text>
					</view>
				</view>
				<view class="article-info-follow">
					<button v-if="userId != article.uId && !followed"  class="followed-btn" @tap="follow">关注</button>
					<button v-if="userId != article.uId && followed" class=" follow-btn cancel" @tap="cancelFollow">取消</button>
				</view>
			</view>
			<view class="grace-text article-content"><rich-text :nodes="article.content" bindtap="tap"></rich-text></view>
			<view class="article-info-collect">
				<button v-if="!collected"  class="collected-btn" @tap="collect">收藏</button>
				<button v-if="collected" class=" collect-btn cancel" @tap="cancelCollect">取消收藏</button>
			</view>
		</view>
		<view class=" article-flow"></view>
		<view class="detail-center">
			<view class="detail-center-box">
				<text class="info-text">评论{{ comments.length }}</text>
				<text class="info-text">按时间倒序</text>
			</view>
		</view>

		<view class="detail-second-box">
			<view class="comment-item" v-for="(comment, index) in comments" :key="index">
				<view class="left"><image :src="comment.avatar" class="small-avatar"></image></view>
				<view class="right">
					<view class="right-nickename">
						<text>{{ comment.nickname }}</text>
					</view>
					<view class=" right-content-box">
						<text>{{ comment.content }}</text>
					</view>
					<view class="right-lou">
						<text>{{ comments.length - index }}楼</text>
						<text>{{ comment.commentTime }}</text>
					</view>
				</view>
			</view>
			<input class="comment-box" type="text" placeholder="写下你的评论" v-model="content" required="required" />
			<button @tap="send">提交</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			article: {
				aId: 0,
				uId: 0,
				title: '',
				content: '',
				avatar: '',
				nickname: '',
				createTime: ''
			},
			comments: [],
			content: '',
			userId: uni.getStorageSync('login_key').userId,
			followed: false,
			collected:false,
		};
	},
	onLoad: function(option) {
		//option为object类型，会序列化上个页面传递的参数
		this.article.aId = option.aId;
	},
	onShow: function() {
		this.getArticle();
	},
	onPullDownRefresh: function() {
		this.getArticle();
	},
	methods: {
		getArticle: function() {
			var _this = this;
			uni.request({
				url: this.apiServer + '/article/' + this.article.aId,
				method: 'GET',
				header: { 'content-type': 'application/x-www-form-urlencoded' },
				data: {
					userId: this.userId
				},
				success: res => {
					// console.log(res.data.data.article);
					_this.article.aId = res.data.data.article.id;
					_this.article.uId = res.data.data.article.uid;
					_this.article.title = res.data.data.article.title;
					_this.article.content = res.data.data.article.content;
					_this.article.nickname = res.data.data.article.nickname;
					_this.article.avatar = res.data.data.article.avatar;
					_this.article.createTime = res.data.data.article.createTime;
					_this.comments = res.data.data.comments;
					_this.article.createTime = this.handleTime(_this.article.createTime)
					for(var i=0;i<_this.comments.length;i++){
						_this.comments[i].commentTime = _this.handleTime(_this.comments[i].commentTime)
					}
					if (res.data.data.followed === '已关注') {
						_this.followed = true;
					}
					if(res.data.data.collected === '已收藏') {
						_this.collected = true;
					}
				},
				complete: function() {
					uni.stopPullDownRefresh();
				}
			});
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
		send: function() {
			console.log('评论人编号：' + this.userId + ',文章编号：' + this.article.aId + '，评论内容：' + this.content);
			uni.request({
				url: this.apiServer + '/comment/add',
				method: 'POST',
				header: { 'content-type': 'application/x-www-form-urlencoded' },
				data: {
					aId: this.article.aId,
					uId: this.userId,
					content: this.content
				},
				success: res => {
					if (res.data.code === 0) {
						uni.showToast({
							title: '评论成功'
						});
						this.getArticle();
						this.content = '';
					}
				}
			});
		},
		follow:function(){
			uni.request({
				url: this.apiServer + '/follow/add',
				method: 'POST',
				header: { 'content-type': 'application/x-www-form-urlencoded' },
				data: {
					fromUId: this.userId,
					toUId: this.article.uId
				},
				success: res => {
					if (res.data.code === 0) {
						uni.showToast({
							title: '关注成功'
						});
						this.followed = true;
					}
				}
			});
		},
		collect:function(){
			uni.request({
				url: this.apiServer + '/collect/add',
				method: 'POST',
				header: { 'content-type': 'application/x-www-form-urlencoded' },
				data: {
					uId: this.userId,
					aId: this.article.aId
				},
				success: res => {
					if (res.data.code === 0) {
						uni.showToast({
							title: '收藏成功'
						});
						this.collected = true;
					}
				}
			});
		},
		cancelFollow:function(){
			uni.request({
				url: this.apiServer + '/follow/cancel',
				method: 'POST',
				header: { 'content-type': 'application/x-www-form-urlencoded' },
				data: {
					fromUId: this.userId,
					toUId: this.article.uId
				},
				success: res => {
					if (res.data.code === 0) {
						uni.showToast({
							title: '已取消关注'
						});
						this.followed = false;
					}
				}
			});
		},
		cancelCollect:function(){
			uni.request({
				url: this.apiServer + '/collect/cancel',
				method: 'POST',
				header: { 'content-type': 'application/x-www-form-urlencoded' },
				data: {
					uId: this.userId,
					aId: this.article.aId
				},
				success: res => {
					if (res.data.code === 0) {
						uni.showToast({
							title: '已取消收藏'
						});
						this.collected = false;
					}
				}
			});
		}
	}
};
</script>

<style>
.container {
	width: 100%;
}
.small-avatar {
	margin-top: 20px;
	width: 50px;
	height: 50px;
	border-radius: 50%;
}
.detail-box {
	width: 92%;
	margin: 0 auto;
	padding-top: 25px;
}
.article-title {
	display: -webkit-box;
	-webkit-box-orient: vertical;
	font-size: 25px;
	font-weight: 800;
	color: #2e2e2e;
}
.article-info-box {
	display: flex;
	justify-content: space-between;
}
button:after {
	border: none;
}
.article-info-follow {
	display: flex;
	align-items: center;
}
.article-info-collect {
	display: flex;
	align-items: center;
}
.comment-box {
	border: 1px solid #fff;
	border-radius: 5px;
	background-color: #eee;
	height: 40px;
}
.followed-btn {
	height: 30x;
	width: 80px;
	font-size: 15px;
	color: #FFFFFF;
	background-color: #E74A39;
	border-radius: 20px;
}
.collected-btn {
	height: 30x;
	width: 100px;
	font-size: 15px;
	color: #FFFFFF;
	background-color: #E74A39;
	border-radius: 20px;
}
.follow-btn {
	height: 30x;
	width: 80px;
	font-size: 15px;
	color: #black;
	background-color: #darkgray;
	border-radius: 20px;
}
.collect-btn {
	height: 30x;
	width: 100px;
	font-size: 15px;
	color: #black;
	background-color: #darkgray;
	border-radius: 20px;
}
.article-info {
	display: flex;
	align-items: center;
	color: #4c4c4c;
	font-size: 15px;
	margin-top: 11px;
}
.article-content {
	color: #4c4c4c;
	font-size: 17px;
	line-height: 28px;
	padding-top: 18px;
}
.detail-center {
	background-color: #f8f8f8;
	display: flex;
	align-items: center;
	height: 45px;
	border-bottom: 1px solid #eeeeee;
}
.detail-center-box {
	width: 92%;
	margin: 0 auto;
	display: flex;
	justify-content: space-between;
	font-size: 15px;
	color: #8f8f94;
}
.detail-second-box {
	width: 92%;
	margin: 0 auto;
}
.comment-item {
	display: flex;
	padding-top: 15px;
	border-bottom: 1px solid #eeeeee;
	padding-bottom: 15px;
}
.right {
	margin-left: 10px;
}
.right2{
	margin-left: 10px;
	margin-top: 15px;
}
.right-nickename {
	color: #2e2e2e;
	font-size: 18px;
}
.right-content-box {
	margin-top: 10px;
	font-size: 15px;
	color: #5e5e5e;
}
.right-lou {
	margin-top: 10px;
	font-size: 12px;
	color: #8f8f94;
}
.right-lou2 {
	margin-top: 10px;
	font-size: 12px;
	color: #8f8f94;
}
.article-flow {
	height: 11px;
	background-color: #f0f0f0;
}

</style>
