<template>
  <div class="article-list">
    <el-table
      :data="article.data.list"
      stripe
      style="width: 100%">
      <el-table-column
        prop="title"
        label="标题">
      </el-table-column>
      <el-table-column
        prop="desc"
        label="描述">
      </el-table-column>
      <el-table-column
        prop="createAt"
        label="创建时间"
        width="170">
      </el-table-column>
      <el-table-column
        prop="updateAt"
        label="更新时间"
        width="170">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <router-link :to="{ name: 'articleDetails' , params : { id: scope.row._id } }">
            <el-button size="small">
                查看
            </el-button>
          </router-link>

          <template v-if="userInfo.isAdmin && userInfo.isAdmin === 1">
            <router-link :to="{ path: '/createdArticle' , query : { id: scope.row._id } }">
              <el-button size="small">
                  更新
              </el-button>
            </router-link>

            <el-button
              size="small"
              type="danger"
              @click="deleteArticle(scope.$index, scope.row)">
              删除
            </el-button>
          </template>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination">
      <el-pagination
        @current-change="pageChange"
        layout="prev, pager, next"
        :page-size="1"
        :current-page="article.currPage"
        :total="article.data.totalPage">
      </el-pagination>
    </div>
  </div>
</template>

<script>
  import {mapGetters, mapActions} from 'vuex';
  export default {
    data() {
      return {
      };
    },
    beforeMount(){
      this.getArticleData();
    },
    mounted(){
    },
    computed: {
      ...mapGetters({
        //article
        article : 'getArticle',
        //global
        userInfo : 'getUserInfo'
      })
    },
    methods: {
      ...mapActions([
        'fetchArticleList',
        'delOneArticle'
      ]),
      getArticleData(){
        let params = {
          categoryName : '全部',
          page : this.article.currPage,
          pageSize : this.$store.state.global.pageSize
        };
        this.fetchArticleList(params);
      },
      pageChange(val){
        //设置页码
        this.$store.commit('setPage', val);
        this.getArticleData();
      },
      //数据为0条时重新请求
      dataReRequested(){
        let articleData = this.article.data;
        let isFirstPage = articleData.firstPage === true;
        let staffIsZero = articleData.list.length === 0;

        if (isFirstPage && staffIsZero) {
          //设置页码
          this.$store.commit('setPage', 1);
          this.getArticleData();
          return;
        }

        if (!isFirstPage && staffIsZero) {
          //设置页码
          this.$store.commit('setPage', articleData.currentPage - 1);
          this.getArticleData();
        }
      },
      //删除文章
      deleteArticle (index , item){
        let self = this;
        const delArticleAction = () => {
          let params = {
            id : item._id,
            index : index
          };

          self.delOneArticle(params)
            .then(res => {
              //数据为0条时重新请求
              self.dataReRequested();

              self.$message({
                type: 'success',
                message: '删除成功!'
              });

            });
        };

        this.$confirm('确认要删除该文章吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {

          delArticleAction();

        });


      }
    }
  };
</script>

<style lang="" rel="stylesheet/">

</style>
