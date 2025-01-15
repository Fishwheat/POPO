<template>
  <view class="main" :style="{
      '--primary-color': themeObj[currentTheme].primary,
      '--border-color': themeObj[currentTheme].border,
      '--border-shadow-color': themeObj[currentTheme].borderShadow,
      '--background-color': themeObj[currentTheme].background,
      '--background-color-delete': themeObj[currentTheme].backgroundDelete,
      '--text-color': themeObj[currentTheme].text,
    }">
    <div class="search">
      <input class="search-input" type="text" v-model="searchValue" placeholder=" " @focus="() => isSearchInputFocus = true" @blur="isSearchInputFocus = false">
      <div class="search-input-placeholder" v-if="!(isSearchInputFocus || searchValue)" :style="{color: themeObj[currentTheme].primary}">请输入商品名称！</div>
      <div class="common-btn category-search">按品类搜索</div>
      <div class="common-btn all-search">全局搜索</div>
    </div>
    <div class="content">
      <div class="left">
        <div class="banner">
          <div class="category-item" v-for="item in categoryList" :key="item.id">
            <div class="name">{{ item.name }}</div>
            <div class="common-btn common-delete delete" v-if="isEdit"></div>
          </div>
        </div>
        <div class="common-btn add-category">+品类</div>
      </div>
      <div class="right">
        <div class="operate">
          <div class="common-btn sort" @click="() => isSortAllow = !isSortAllow">
            排序
            <div class="sort-list" :class="[{ 'sort-list-allow': isSortAllow }]">
              <div class="sort-list-item" v-for="item in selectedSortTypeOptions" :key="item.value" @click.stop="handleSort(item)">
                {{ item.label }}
              </div>
            </div>
          </div>
          <div class="common-btn edit" @click="handleEdit()">{{ !isEdit ? '编辑' : '保存' }}</div>
          <div class="common-btn theme" @click="() => isThemeAllow = !isThemeAllow">
            主题
            <div class="theme-list" :class="[{ 'theme-list-allow': isThemeAllow }]">
              <div class="theme-list-item" v-for="item in themeOptions" :key="item.value" @click.stop="handleTheme(item)">
                {{ item.label }}
              </div>
            </div>
          </div>
        </div>
        <div class="banner">
          <div class="commodity-card" v-for="item in categoryList[0].children" :key="item.id">
            <div class="common-btn common-delete delete" v-if="isEdit"></div>
            <div class="name">{{ item.name }}</div>
            <div class="bottom-info">
              <div class="price">{{ item.price }}￥</div>
              <div class="count-cover">
                <div class="common-btn decrease">-</div>
                <div class="count">
                  <input v-if="isEdit" type="text" v-model="commodityCount">
                  <span v-else>{{ commodityCount }}</span>
                </div>
                <div class="common-btn increase">+</div>
              </div>
            </div>
          </div>
        </div>
        <div class="common-btn add-commodity">+产品</div>
      </div>
    </div>
  </view>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'

const selectedSortTypeOptions = [
  { label: "按添加时间排序", value: 'time' },
  { label: "按库存排序", value: 'num' },
]
const themeOptions = [
  {
    label: '浅粉色',
    value: 'pink',
  },
  {
    label: '浅绿色',
    value: 'green',
  },
  {
    label: '浅紫色',
    value: 'purple',
  },
  {
    label: '深黑色',
    value: 'black',
  },
  {
    label: '白色',
    value: 'white',
  }
]
const themeObj = {
  pink: {
    primary: '#f8bbd0',
    border: '#fad1e0',
    borderShadow: '#f1e3e8',
    background: '#f8bbd0',
    backgroundDelete: '#880835',
    text: '#fff',
  },
  green: {
    primary: '#d4edda',
    border: '#e9f1eb',
    borderShadow: '#e7f0e9',
    background: '#d4edda',
    backgroundDelete: '#094417',
    text: '#fff',
  },
  purple: {
    primary: '#f3e5f5',
    border: '#f4eef5',
    borderShadow: '#f6f3f7',
    background: '#f3e5f5',
    backgroundDelete: '#671875',
    text: '#fff',
  },
  black: {
    primary: '#2c3e50',
    border: '#000',
    borderShadow: '#b3bac2',
    background: '#2c3e50',
    backgroundDelete: '#000',
    text: '#000',
  },
  white: {
    primary: '#333',
    border: '#dcdfe6',
    borderShadow: '#ebe5e5',
    background: '#fff',
    backgroundDelete: '#fff',
    text: '#333',
  },
}

const isEdit = ref(false)
const searchValue = ref('')
const isSearchInputFocus = ref(false)
const selectedSortType = ref('time')
const commodityCount = ref(0)
const isSortAllow = ref(false)
const currentSort = ref('time')
const isThemeAllow = ref(false)
const currentTheme = ref('pink')
const categoryList = ref(Array.from({ length: 20 }, (_, i) => ({
  name: '品类'+i,
  id: i,
  children: Array.from({ length: 20 }, (_, i) => ({
    name: '德莱文的斧子'+i,
    price: '798',
    id: '123' + i
  }))
})))

const handleEdit = () => {
  isEdit.value = !isEdit.value
}
const sortTypeChange = (val: string) => {
  console.log('sortTypeChange', val);
}
const handleSort = (params: { label: string; value: string }) => {
  currentSort.value = params.value
  isSortAllow.value = false
}
const handleTheme = (params: { label: string; value: string }) => {
  currentTheme.value = params.value
  isThemeAllow.value = false
}

</script>

<style lang="scss">
.common-btn {
  text-align: center;
  color: var(--text-color);
  background: var(--background-color);
  border: 1px solid var(--border-color);
  border-radius: 5px;
  padding: 5px 10px;
  cursor: pointer;
}
.common-delete {
  position: relative;
  width: 10px;
  height: 10px;
  background: var(--background-color-delete);
  padding: 2px;
  &::after {
    content: 'x';
    position: absolute;
    top: 0;
    right: 0;
    width: 100%;
    height: 100%;
    text-align: center;
    line-height: 12px;
    color: var(--text-color);
  }
}

// input::placeholder {
//   color: var(--primary-color); /* 设置 placeholder 字体颜色 */
//   font-size: 14px; /* 可选：设置字体大小 */
//   font-style: italic; /* 可选：设置字体样式 */
// }

// input::-webkit-input-placeholder { /* Chrome, Safari, Opera */
//   color: var(--primary-color);
// }

// input::-moz-placeholder { /* Firefox 19+ */
//   color: var(--primary-color);
// }

// input:-ms-input-placeholder { /* IE 10+ */
//   color: var(--primary-color);
// }

// input::placeholder { /* 适用于大多数现代浏览器 */
//   color: var(--primary-color);
// }

.main {
  width: 100vw;
  height: 100vh;
  font-size: 16px;
  .search {
    position: relative;
    display: flex;
    align-items: center;
    width: calc(100% - 10px);
    height: 40px;
    border: 1px solid var(--border-color);
    border-radius: 10px;
    box-sizing: border-box;
    margin: 0 0 5px 5px;
    padding: 0 5px;
    .search-input {
      flex: 1;
      color: var(--primary-color);
    }
    .search-input-placeholder {
      position: absolute;
      top: 50%;
      left: 5px;
      transform: translateY(-50%);
      pointer-events: none;
    }
    .category-search {
      margin: 0 5px;
    }
    .all-search {}
  }
  .content {
    display: flex;
    width: 100%;
    height: calc(100% - 50px);
    padding: 0 5px;
    box-sizing: border-box;
    .left {
      position: relative;
      width: 40%;
      height: 100%;
      border: 1px solid var(--border-color);
      border-radius: 10px;
      box-sizing: border-box;
      .banner {
        width: 100%;
        height: 100%;
        overflow: auto;
        padding: 5px;
        padding-bottom: 45px;
        box-sizing: border-box;
        .category-item {
          position: relative;
          width: 100%;
          border-radius: 5px;
          border: 1px solid var(--border-shadow-color);
          box-shadow: 0px 1px 0px 1px var(--border-shadow-color);
          box-sizing: border-box;
          padding: 5px;
          .name {
            color: var(--primary-color);
            padding-right: 20px;
          }
          .delete {
            position: absolute;
            top: 7px;
            right: 5px;
          }
          &+.category-item {
            margin-top: 5px;
          }
        }
      }
      .add-category {
        position: absolute;
        right: 5px;
        bottom: 5px;
      }
    }
    .right {
      position: relative;
      width: calc(60% - 2px);
      height: 100%;
      border: 1px solid var(--border-color);
      border-radius: 10px;
      box-sizing: border-box;
      margin-left: 2px;
      .operate {
        width: 100%;
        height: 44px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        box-sizing: border-box;
        padding: 5px;
        .sort {
          position: relative;
          width: 40px;
          .sort-list {
            position: absolute;
            left: 0;
            top: 33px;
            height: auto;
            border-radius: 5px;
            background: var(--background-color);
            box-sizing: border-box;
            display: none;
            z-index: 10;
            .sort-list-item {
              text-align: left;
              color: var(--text-color);
              white-space: nowrap;
              padding: 5px;
            }
          }
          .sort-list-allow {
            display: block;
          }
        }
        .edit {
          width: 40px;
        }
        .theme {
          position: relative;
          width: 40px;
          .theme-list {
            position: absolute;
            left: 0;
            top: 33px;
            height: auto;
            border-radius: 5px;
            background: var(--background-color);
            box-sizing: border-box;
            display: none;
            z-index: 10;
            .theme-list-item {
              text-align: left;
              color: var(--text-color);
              white-space: nowrap;
              padding: 5px;
            }
          }
          .theme-list-allow {
            display: block;
          }
        }
      }
      .banner {
        width: 100%;
        height: calc(100% - 44px);
        overflow: auto;
        padding: 5px;
        padding-bottom: 45px;
        box-sizing: border-box;
        .commodity-card {
          position: relative;
          width: 100%;
          border-radius: 5px;
          border: 1px solid var(--border-shadow-color);
          box-shadow: 0px 1px 0px 1px var(--border-shadow-color);
          box-sizing: border-box;
          padding: 5px;

          &+.commodity-card {
            margin-top: 5px;
          }
          
          .delete {
            position: absolute;
            top: 7px;
            right: 5px;
          }
          .name {
            color: var(--primary-color);
            padding-right: 20px;
          }
          .bottom-info {
            display: flex;
            justify-content: space-between;
            margin-top: 10px;
            .price {
              color: var(--primary-color);
              margin-right: 5px;
            }
            .count-cover {
              flex: 1;
              display: flex;
              align-items: center;
              justify-content: flex-end;
              .decrease {
                width: 20px;
                min-width: 20px;
                height: 20px;
                line-height: 16px;
                padding: 0;
                box-sizing: border-box;
              }
              .count {
                color: var(--primary-color);
                margin: 0 5px;
              }
              .increase {
                width: 20px;
                min-width: 20px;
                height: 20px;
                line-height: 16px;
                padding: 0;
                box-sizing: border-box;
              }
            }
          }
        }
      }
      .add-commodity {
        position: absolute;
        right: 5px;
        bottom: 5px;
      }
    }
  }
}
</style>
