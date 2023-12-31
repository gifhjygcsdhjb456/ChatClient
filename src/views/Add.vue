<template>
  <div class="add-page">
    <div class="header">
      <div class="search-area">
        <el-input
          placeholder="请输入内容"
          prefix-icon="el-icon-search"
          v-model="searchKey"
          maxlength="20"
          @input="keyWordChange"
        >
        </el-input>
        <el-select v-model="searchObject" placeholder="请选择搜索对象" @change="fetch(false, true)">
          <el-option v-for="item in searchObjectMap" :key="item.id" :label="item.label" :value="item.value" />
        </el-select>
        <el-select v-model="searchType" placeholder="请选择" @change="fetch(false, true)">
          <el-option v-for="item in searchTypes[searchObject]" :key="item.id" :label="item.label" :value="item.value" />
        </el-select>
      </div>
      <div class="condition-list">
        <el-tag type="danger">当前搜索条件：</el-tag>
        <el-tag v-if="searchKey" closable @close="handleTagClose">{{ searchKey }}</el-tag>
        <el-tag v-if="searchObject">{{ searchObject }}</el-tag>
        <el-tag v-if="searchType">{{ searchType }}</el-tag>
      </div>
    </div>
    <div class="body">
      <div class="search-outcome" v-loading="isFetch">
        <div class="description" v-if="searchList.length">一共搜索到{{ searchList.length }}条结果</div>
        <empty-svg class="no-data" v-if="searchList.length === 0" width="200" height="200" />
        <user-list :searchList="searchList" v-if="searchObject === 'friend'" />
        <group-list :searchList="searchList" v-else-if="searchObject === 'group'" />
        <div class="no-more" v-if="searchList.length && !hasMore">
          <el-alert title="没有更多了..." type="info" center show-icon :closable="false" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { debounce } from '@/utils'
import { searchObjectMap, searchTypes } from '@/const'
import userList from '@/components/customSearchList/userList'
import groupList from '@/components/customSearchList/groupList'
import EmptySvg from '@/SVGComponents/empty'

export default {
  name: 'Add',
  data() {
    return {
      searchKey: '',
      searchObjectMap,
      searchTypes,
      searchObject: 'friend',
      searchType: 'code',
      isFetch: false,
      searchList: [],
      hasMore: true,
      pageSize: 7,
      pageIndex: 0
    }
  },
  methods: {
    async fetchUser(params, loadMore) {
      try {
        const { data } = await this.$http.preFetchUser(params)
        if (data.code === 2000) {
          const userList = data.data.userList
          // console.log("返回查询的用户列表为：", userList)
          if (loadMore) {
            this.searchList = [...this.searchList, ...userList]
          } else {
            this.searchList = userList
          }
          if (userList.length < 7) {
            this.hasMore = false
          } else {
            this.hasMore = true
          }
        }
      } catch (error) {
      } finally {
        this.isFetch = false
      }
    },
    async fetchGroup(params, loadMore) {
      try {
        const { data } = await this.$http.preFetchGroup(params)
        if (data.code === 2000) {
          const groupList = data.data.groupList
          // console.log("返回查询的群列表为：", groupList)
          groupList.forEach((item) => {
            item.isGroup = true
          })
          if (loadMore) {
            this.searchList = [...this.searchList, ...groupList]
          } else {
            this.searchList = groupList
          }
          if (groupList.length < 7) {
            this.hasMore = false
          } else {
            this.hasMore = true
          }
        }
      } catch (error) {
      } finally {
        this.isFetch = false
      }
    },
    handleTagClose() {
      this.searchKey = ''
      this.fetch()
    },
    fetch(loadMore = false, initPage = false) {
      if (!this.searchKey) {
        this.isFetch = true
        setTimeout(() => {
          this.searchList = []
          this.isFetch = false
        }, 500)
        return
      }
      if (initPage) {
        this.pageIndex = 0
        this.hasMore = true
      }
      this.isFetch = true
      const params = {
        type: this.searchType,
        searchContent: this.searchKey,
        pageIndex: this.pageIndex,
        pageSize: this.pageSize
      }
      if (this.searchObject === 'friend') {
        this.fetchUser(params, loadMore)
      } else if (this.searchObject === 'group') {
        this.fetchGroup(params, loadMore)
      }
    },
    keyWordChange: debounce(function () {
      this.fetch(false)
    }, 500),
    handlerScroll: debounce(function () {
      const addPage = document.querySelector('.add-page')
      const scrollTop = addPage.scrollTop
      const clientHeight = addPage.clientHeight
      const scrollHeight = addPage.scrollHeight
      if (scrollTop + clientHeight + 10 >= scrollHeight) {
        if (this.hasMore) {
          this.pageIndex++
          this.fetch(true)
        }
      }
    }, 500)
  },
  components: {
    userList,
    groupList,
    EmptySvg
  },
  mounted() {
    const addPage = document.querySelector('.add-page')
    addPage.addEventListener('scroll', this.handlerScroll)
  },
  watch: {
    searchObject(newVal) {
      this.searchType = 'code'
    }
  }
}
</script>

<style lang="scss">
.add-page {
  width: 815px;
  height: 100%;
  padding: 10px 0;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  margin: 0px auto;
  // margin-left: 65px;

  .header {
    .search-area {
      display: flex;
    }

    .condition-list {
      margin-top: 17px;
    }
  }

  .body {
    margin-top: 17px;
    flex: 1;
    width: 815px;
    .search-outcome {
      position: relative;
      background-color: #fff;
      border-radius: 3px;
      width: 100%;
      height: 100%;
      padding: 10px;

      .description {
        margin-bottom: 10px;
      }

      .no-data {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }

      .no-more {
        margin-top: 10px;
      }
    }
  }
}
</style>
