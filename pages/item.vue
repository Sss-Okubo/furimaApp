<template>
  <v-app id="app">
    <v-container>
      <div class="main">
        <div class="item-image">
          <v-carousel>
            <v-carousel-item v-for="image in images" :key="image.id">
              <v-img :src=image.url aspect-ratio="1"></v-img>
            </v-carousel-item>
          </v-carousel>
        </div>
        <div class="item-detail">
          <v-sheet border="md" class="pa-6 text-white mx-auto overflow-y-auto" width="100%" max-height="500"
            max-width="500">
            <h4 class="text-h5 font-weight-bold mb-2">商品名</h4>
            <p class="mb-4">
              {{ goods.goods_name }}
            </p>
            <div class="item-price align-end">
              <p class="text-h5 font-weight-bold">{{ price }}</p>&nbsp;
              <p class="text-body-2 ">(税込)送料込み</p>
            </div>
            <div class="item-opinion  mb-4">
              <div v-if="islogin" class="item-like">
                <v-btn v-if="islike" @click="deleteLike(goods.id)" icon color="yellow">
                  <v-icon large>mdi-star</v-icon>
                </v-btn>
                <v-btn v-else @click="insertLike(goods.id)" icon color="Glay">
                  <v-icon large>mdi-star-outline</v-icon>
                </v-btn><br>&nbsp;
                <p class="text-caption" name="likecount">{{ likes.likeCount }}</p>
              </div>
              <div class="item-comment ml-4">
                <v-btn @click="scrollToSection()" icon color="glay">
                  <v-icon large>mdi-chat-outline</v-icon>
                </v-btn><br>&nbsp;
                <p class="text-caption" name="commentcount">{{ comments.length }}</p>
              </div>
            </div>
            <div>
              <v-btn block class="text-none text-black mb-4" color="error" dark size="x-large" variant="flat"
                v-on:click="clickBuy(goods.id)">
                購入する
              </v-btn>
            </div>
            <div class="item-explane">
              <h4 class="text-h6 font-weight-bold ">商品の説明</h4>

              <p class="mb-4 text-body-2">
                {{ goods.detail }}
              </p>
            </div>
            <div class="item-info">
              <h4 class="text-h6 font-weight-bold ">商品の情報</h4>

              <div class="item-category mb-4">
                <div class="text-lg-subtitle-1 font-weight-bold">
                  カテゴリ
                </div>
                <div v-for="(category) in this.categories">
                  <v-chip small class="ml-2" color="#80DEEA" dark>
                    {{ category.value }}
                  </v-chip>
                </div>
              </div>

              <div class="item-condition mb-4">
                <div class="text-lg-subtitle-1 font-weight-bold">
                  商品の状態
                </div>
                <v-chip small class="ml-2" color="#80DEEA" label dark>
                  {{ goods.value }}
                </v-chip>
              </div>
            </div>
            <div class="comment" id="comment">
              <h4 class="text-lg-subtitle-1 font-weight-bold ">コメント</h4>
              <v-card tile outlined class="mx-auto" v-for="(comment, index) in this.comments">
                <v-list three-line>
                  <v-list-item :key="comment.id">
                    <v-list-item-avatar>
                      <v-img v-if="comment.url" :src="comment.url" contain></v-img>
                    </v-list-item-avatar>
                    <v-list-item-content>
                      <v-list-item-title v-html="comment.name"></v-list-item-title>
                      <v-list-item-subtitle v-html="comment.comment"></v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                  <v-btn v-if="comment.user_id === loginUserId" icon @click="deleteComment(comment.id)" class="ml-3">
                    <v-icon color="red">mdi-delete</v-icon>削除
                  </v-btn>
                </v-list>
              </v-card>
              <v-text-field v-if="islogin" label="商品へのコメント" placeholder="" v-model="newComment"
                color="red"></v-text-field>
              <v-btn v-if="islogin" block class="mb-4" color="red" dark size="x-large" variant="flat"
                v-on:click="insertComment(goods.id)">
                コメントを送信する
              </v-btn>
              <v-divider></v-divider>
            </div>
          </v-sheet>
        </div>
      </div>
    </v-container>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      loginUserId: "",
      islike: false,
      islogin : false,
      goods: [],
      images: [],
      likes:[],
      price: "",
      comments: [],
      newComment: "",
      categories: [],
      imageUrl: null,
      goodsId:""
    };
  
  },
  
  methods: {
    // 商品取得（id)
    async getGoodsById(goodsId) {
      const resData = await this.$axios.get(
        "http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/goods/" + goodsId
      );
      this.goods = resData.data.data[0];
      this.price = this.goods.price.toLocaleString('ja-JP',{style:'currency',currency:'JPY'});
    },

    // 画像取得
    async getImages(goodsId) {
      const resData = await this.$axios.get(
        "http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/image/" + goodsId
      );
      this.images = resData.data.data;
    },

    // お気に入り取得
    async getLikes(goodsId, loginID) {
      const resData = await this.$axios.get(
        "http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/likes/" + goodsId + '/' + loginID
      );
      this.likes = resData.data.data;
      this.islike = this.likes.isLike;
    },
    // liked
    async insertLike(id) {
      const sendData = {
          goods_id: id,
          user_id: this.loginUserId,
        };
        await this.$axios.post("http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/likes/", sendData);
        this.getLikes(id, this.loginUserId);
    },
    // unliked
    async deleteLike(id) {
      await this.$axios.delete("http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/likes/" + id + '/' + this.loginUserId);
      this.getLikes(id, this.loginUserId);
    },
    // コメント削除
    async deleteComment(id) {
      await this.$axios.delete("http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/comments/" + id);
      this.getComments(this.goodsId);
    },

    // コメント取得
    async getComments(goodsId) {
      const resData = await this.$axios.get(
        "http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/comments/" + goodsId 
      );
      this.comments = resData.data.data;
    },

    // コメント登録
    async insertComment(goodsId) {
      const sendData = {
        goods_id: goodsId,
        user_id: this.loginUserId,
        comment: this.newComment,
      };
      await this.$axios.post("http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/comments/", sendData);
      this.getComments(goodsId);
            this.newComment = "";
    },

    // 指定した要素までスクロール
    scrollToSection() {
      const container = document.querySelector('.pa-6');
      const el = document.getElementById('comment');
      this.$vuetify.goTo(el, { container: container });
    },

    // カテゴリ取得
    async getCategories(goodsId) {
      const resData = await this.$axios.get(
        "http://ec2-35-72-4-140.ap-northeast-1.compute.amazonaws.com/api/categories/" + goodsId
      );
      this.categories = resData.data.data;
    },
    clickBuy(goodsId) {
    // 購入ボタン押下時
      this.$router.push({ path: '/purchase', query: { itemId: goodsId, userId: this.loginUserId } });
    },
  },
  created() {
    this.goodsId = this.$route.query.itemId;
    this.loginUserId = parseInt(this.$route.query.userId);
    this.getGoodsById(this.goodsId);
    this.getImages(this.goodsId);
    this.getLikes(this.goodsId, this.loginUserId);
    this.getComments(this.goodsId, this.loginUserId);
    this.getCategories(this.goodsId);
    if (this.loginUserId) {
      this.islogin = true;
    }
  },
};
</script>

<style>
.main {
  display: flex;
  margin-top: 1rem;
}
.item-image {
  padding: 15px;
  width: 50%;
  background-color: #fff;
  outline: #fff;
}
.item-detail {
  padding: 15px;
  width: 50%;
  background-color: #fff;
  outline: #fff;
}
.item-price{
  display: flex;
}
.item-opinion{
display: flex;
}
.item-condition{
  display: flex;
}
.item-category{
  display: flex;
}
.text-h6 {
  color: rgb(105, 105, 105);
}
.text-lg-subtitle-1{
  color: rgb(105, 105, 105);;
}
.text-body-2{
  color: rgb(105, 105, 105);;
}
@media not all and (min-width: 768px) {
  .item {
    width: 100%;
  }

  .item:not(:last-child) {
    margin-bottom: 30px;
  }
}
</style>