<template>
  <div class="me-view-comment-item">
    <div class="me-view-comment-author">
      <a class="">
        <img class="me-view-picture" :src="comment.author.avatar"></img>
      </a>
      <div class="me-view-info">
        <span class="me-view-nickname">{{comment.author.nickname}}</span>
        <div class="me-view-meta">
          <span>{{rootCommentCounts - index}}樓</span>
          <span>{{comment.createDate | format}}</span>
        </div>
      </div>
    </div>
    <div>
      <p class="me-view-comment-content">{{comment.content}}</p>
      <div class="me-view-comment-tools">
        <!--<a class="me-view-comment-tool">-->
        <!--<i class="el-icon-caret-top"></i> 20-->
        <!--</a>-->
        <a class="me-view-comment-tool" @click="showComment(-1,comment.author)">
          <i class="me-icon-comment"></i>&nbsp; 評論
        </a>
      </div>

      <div class="me-reply-list">
        <div class="me-reply-item" v-for="c in comment.childrens" :key="c.id">
          <div style="font-size: 14px">
            <span class="me-reply-user">{{c.author.nickname}}:&nbsp;&nbsp;</span>

            <span v-if="c.level == 2" class="me-reply-user">@{{c.toUser.nickname}} </span>

            <span>{{c.content}}</span>
          </div>
          <div class="me-view-meta">
            <span style="padding-right: 10px">{{c.createDate | format}}</span>
          <!--  <a class="me-view-comment-tool" @click="showComment(c.id, c.author)">
              <i class="me-icon-comment"></i>&nbsp;回覆
            </a> -->
          </div>

        </div>

        <div class="me-view-comment-write" v-show="commentShow">

          <el-input
            v-model="reply.content"
            type="input"
            style="width: 90%"
            :placeholder="placeholder"
            class="me-view-comment-text"
            resize="none">
          </el-input>

          <el-button style="margin-left: 8px" @click="publishComment()" type="text">評論</el-button>

        </div>

      </div>

    </div>
  </div>
</template>

<script>
  import {publishComment} from '@/api/comment'

  export default {
    name: "CommentItem",
    props: {
      articleId: Number,
      comment: Object,
      index: Number,
      rootCommentCounts: Number
    },
    data() {
      return {
        placeholder: '你的評論...',
        commentShow: false,
        commentShowIndex: '',
        reply: this.getEmptyReply()
      }
    },
    methods: {
      showComment(commentShowIndex, toUser) {
        this.reply = this.getEmptyReply()

        if (this.commentShowIndex !== commentShowIndex) {
          if (toUser) {
            this.placeholder = `@${toUser.nickname} `
            this.reply.toUserId = toUser.id
          } else {
            this.placeholder = '你的評論...'
          }

          this.commentShow = true
          this.commentShowIndex = commentShowIndex
        } else {
          this.commentShow = false
          this.commentShowIndex = ''
        }
      },
      publishComment() {
        let that = this
        if (!that.reply.content) {
          return;
        }

        publishComment(that.reply,this.$store.state.token).then(data => {
          if(data.success){
            that.$message({type: 'success', message: '評論成功', showClose: true})
            if(!that.comment.childrens){
              that.comment.childrens = []
            }
            that.comment.childrens.unshift(data.data)
            that.$emit('commentCountsIncrement')
            that.showComment(that.commentShowIndex)
          }else{
             that.$message({type: 'error', message: data.msg, showClose: true})
          }
        }).catch(error => {
          if (error !== 'error') {
            that.$message({type: 'error', message: '評論失敗', showClose: true})
          }
        })

      },
      getEmptyReply() {
        return {
          articleId: this.articleId,
          parent: this.comment.id,
          toUserId: '',
          content: ''
        }
      }
    }
  }
</script>

<style>
  .me-view-tag-item {
    margin: 0 4px;
  }

  .me-view-comment {
    margin-top: 60px;
  }

  .me-view-comment-title {
    font-weight: 600;
    border-bottom: 1px solid #f0f0f0;
    padding-bottom: 20px;
  }

  .me-view-comment-write {
    margin-top: 20px;
  }

  .me-view-comment-text {
    font-size: 16px;
  }

  .v-show-content {
    padding: 8px 25px 15px 30px !important;
  }

  .v-note-wrapper .v-note-panel {
    box-shadow: none !important;
  }

  .me-view-comment-item {
    margin-top: 20px;
    padding-bottom: 16px;
    border-bottom: 1px solid #f0f0f0;
  }

  .me-view-comment-author {
    margin: 10px 0;
    vertical-align: middle;
  }

  .me-view-nickname {
    font-size: 14px;
  }

  .me-view-comment-content {
    line-height: 1.5;
  }

  .me-view-comment-tools {
    margin-top: 4px;
    margin-bottom: 10px;
  }

  .me-view-comment-tool {
    font-size: 13px;
    color: #a6a6a6;
    padding-right: 14px;
  }

  .v-note-wrapper .v-note-panel .v-note-show .v-show-content, .v-note-wrapper .v-note-panel .v-note-show .v-show-content-html {
    background: #fff !important;
  }

  .me-reply-list {
    padding-left: 16px;
    border-left: 4px solid #c5cac3;
  }

  .me-reply-item {
    margin-bottom: 8px;
    border-bottom: 1px solid #f0f0f0;
  }

  .me-reply-user {
    color: #78b6f7;
  }
</style>
