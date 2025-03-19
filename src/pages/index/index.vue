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
    </div>
    <div class="search-btn">
      <div class="common-btn category-search" @click="categorySearch">按品类搜索</div>
      <div class="common-btn all-search" @click="allSearch">全局搜索</div>
      <div class="common-btn" @click="cancelSearch">取消</div>
      <div class="search-btn-right">
        <div class="btn-line"></div>
        <div class="common-btn" @click="exportClick">导出</div>
        <div class="common-btn" @click="injectClick">导入</div>
      </div>
    </div>
    <div class="content">
      <div class="left" :class="{'allow-left': isAllowLeft, 'allow-right-left': isAllowRight}">
        <div class="banner">
          <template v-if="data.categoryList?.length">
            <div
              class="category-item"
              @click="categoryItemClick(index)"
              :class="{'category-item-active': index === currentCategoryIndex}"
              v-for="(item, index) in data.categoryList"
              :key="item.id"
            >
              <div class="name">{{ item.name }}</div>
              <div class="common-btn common-delete delete" v-if="isEdit" @click.stop="deleteCategory(item.id)"></div>
            </div>
          </template>
          <div v-else class="common-empty-text">请先新增品类！</div>
        </div>
        <div class="category-allow" @click="allowCategory"></div>
        <div class="common-btn add-category" @click="addCategory">+品类</div>
      </div>
      <div class="right" :class="{'allow-right': isAllowLeft, 'allow-right-right': isAllowRight}">
        <div class="operate">
          <div class="common-btn sort" @click="openSortClick">
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
          <template v-if="commodityListComp?.length">
            <div class="commodity-card" v-for="item in commodityListComp" :key="item.id">
              <div class="common-btn common-delete delete" v-if="isEdit" @click.stop="deleteCommodity(item.id)"></div>
              <div class="name">{{ item.name }}</div>
              <div class="bottom-info">
                <div class="price">{{ item.price }}￥</div>
                <div class="count-cover">
                  <div class="common-btn decrease" @click="decrease(item.id)">-</div>
                  <div class="count">
                    <input v-if="isEdit" type="text" v-model="item.count">
                    <span v-else>{{ item.count }}</span>
                  </div>
                  <div class="common-btn increase" @click="increase(item.id)">+</div>
                </div>
              </div>
            </div>
          </template>
          <div v-else-if="isSearchCommodityList" class="common-empty-text">搜索不到指定产品！</div>
          <div v-else-if="data.categoryList?.length" class="common-empty-text">该品类暂无产品！</div>
          <div v-else class="common-empty-text">请先新增品类！</div>
        </div>
        <div class="category-allow-right" @click="allowRightCategory"></div>
        <div class="common-btn add-commodity" @click="addCommodity">+产品</div>
      </div>
    </div>
    <!-- 新增品类 -->
    <uni-popup ref="addCategoryDialog" type="dialog" >
      <uni-popup-dialog
        type="bottom"
        :mask-click="false"
        mode="input"
        title="新增品类"
        placeholder="请输入品类名"
        @confirm="addCategoryDialogInputConfirm"
      >
      </uni-popup-dialog>
    </uni-popup>
    <!-- 新增产品 -->
    <uni-popup ref="addCommodityDialog" type="dialog" >
      <uni-popup-dialog
        type="bottom"
        :mask-click="false"
        title="新增产品"
        @confirm="addCommodityDialogConfirm"
      >
        <div>
          <div>
            产品名称:
            <input placeholder="名称" v-model="commodityForm.name" class="common-input" />
          </div>
          <div>
            产品价格:
            <input placeholder="价格" v-model="commodityForm.price" class="common-input" />
          </div>
          <div>
            产品数量:
            <input placeholder="数量" v-model="commodityForm.count" class="common-input" />
          </div>
        </div>
      </uni-popup-dialog>
    </uni-popup>
    <!-- 导入数据 -->
    <uni-popup ref="injectDataDialog" type="dialog" >
      <uni-popup-dialog
        type="bottom"
        :mask-click="false"
        mode="input"
        title="导入数据（请确认好数据，将会全面覆盖当前已有数据！）"
        placeholder="请粘贴数据！"
        @confirm="injectDataDialogInputConfirm"
      >
      </uni-popup-dialog>
    </uni-popup>
  </view>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, ref } from 'vue'

interface CommodityType {
  name: string;
  price: string;
  id: string;
  count: string;
  time: number;
}
interface DataType {
  categoryList: {
    id: string;
    name: string
    commodityList: CommodityType[]
  }[]
}

const selectedSortTypeOptions = [
  { label: "按添加时间排序", value: 'time' },
  { label: "按库存排序", value: 'count' },
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

const data = ref<DataType>({
  categoryList: [],
})
const isEdit = ref(false)
const searchValue = ref('')
const isSearchInputFocus = ref(false)
const currentCategoryIndex = ref(0)
const isSortAllow = ref(false)
const currentSort = ref('time')
const isThemeAllow = ref(false)
const currentTheme = ref('pink')
const addCategoryDialog = ref<any>()
const addCommodityDialog = ref<any>()
const injectDataDialog = ref<any>()
const commodityForm = ref({
  name: '',
  price: '',
  count: '',
})
const searchCommodityList = ref<CommodityType[]>([])
const isSearchCommodityList = ref(false)
const isAllowLeft = ref(false)
const isAllowRight= ref(false)

const commodityListComp = computed(() => isSearchCommodityList.value ? searchCommodityList.value : data.value?.categoryList?.[currentCategoryIndex.value]?.commodityList)

const categorySearch = () => {
  if (!data.value?.categoryList?.length) {
    return
  }
  isSearchCommodityList.value = true
  searchCommodityList.value = data.value.categoryList![currentCategoryIndex.value]!.commodityList.filter(item => item.name.includes(searchValue.value))
}
const allSearch = () => {
  if (!data.value?.categoryList?.length) {
    return
  }
  isSearchCommodityList.value = true
  const arr = data.value.categoryList.flatMap(item => item.commodityList)
  searchCommodityList.value = arr.filter(item => item.name.includes(searchValue.value))
}
const cancelSearch = () => {
  isSearchCommodityList.value = false
  searchCommodityList.value = []
  searchValue.value = ''
}

const exportClick = async () => {
  const res = await getStorage()
  if (res?.data) {
    uni.setClipboardData({
      data: res?.data,
      // success: function () {
      //   console.log('success');
      // }
    });
  }
}
const injectClick = (val?: string) => {
  injectDataDialog.value?.open();
}
const injectDataDialogInputConfirm = (val?: string) => {
  setStorage(val)
  getData()
}

const categoryItemClick = (index: number) => {
  currentCategoryIndex.value = index
}
const addCategory = () => {
  addCategoryDialog.value?.open();
}
const addCategoryDialogInputConfirm = async(val?: string) => {
  if (val?.trim()) {
    data.value.categoryList.unshift({
      id: generateUniqueId(),
      name: val.trim(),
      commodityList: []
    })
    setStorage(JSON.stringify(data.value))
    await nextTick()
    currentCategoryIndex.value = 0
  }
}
const deleteCategory = (id: string) => {
  data.value.categoryList = data.value.categoryList.filter(item => item.id !== id)
}

const addCommodity = () => {
  commodityForm.value = {
    name: '',
    price: '',
    count: '',
  }
  addCommodityDialog.value?.open();
}
const addCommodityDialogConfirm = async() => {
  const { name, price, count } = commodityForm.value
  if (!name || !price || !count) {
    return
  }
  data.value.categoryList[currentCategoryIndex.value].commodityList.unshift({
    id: generateUniqueId(),
    name,
    price,
    count: (Number(count) <= 0 ? 0 : count).toString(),
    time: +new Date()
  })
  setStorage(JSON.stringify(data.value))
}
const deleteCommodity = (id: string) => {
  data.value.categoryList![currentCategoryIndex.value]!.commodityList = data.value.categoryList?.[currentCategoryIndex.value]?.commodityList?.filter(item => item.id !== id)
}
const decrease = (id: string) => {
  if (isSearchCommodityList.value) {
    return
  }
  const itemInfo = data.value.categoryList?.[currentCategoryIndex.value]?.commodityList?.find(item => item.id === id)
  const count = Number(itemInfo?.count ?? 0)
  if (count !== 0) {
    itemInfo!.count = (count - 1).toString()
  }
  setStorage(JSON.stringify(data.value))
}
const increase = (id: string) => {
  if (isSearchCommodityList.value) {
    return
  }
  const itemInfo = data.value.categoryList?.[currentCategoryIndex.value]?.commodityList?.find(item => item.id === id)
  itemInfo!.count = (Number(itemInfo?.count ?? 0) + 1).toString()
  setStorage(JSON.stringify(data.value))
}

const openSortClick = () => {
  if (isSearchCommodityList.value) {
    return
  }
  isSortAllow.value = !isSortAllow.value
}
const handleEdit = () => {
  if (isSearchCommodityList.value) {
    return
  }
  isEdit.value = !isEdit.value
  setStorage(JSON.stringify(data.value))
  if (!isEdit.value) {
    currentCategoryIndex.value = 0
  }
}
const handleSort = (params: { label: string; value: string }) => {
  currentSort.value = params.value
  data.value.categoryList?.[currentCategoryIndex.value]?.commodityList?.sort((a, b) => Number(b[currentSort.value as 'count' | 'time']) - Number(a[currentSort.value as 'count' | 'time']))
  isSortAllow.value = false
}
const handleTheme = (params: { label: string; value: string }) => {
  currentTheme.value = params.value
  isThemeAllow.value = false
}

const allowCategory = () => {
  isAllowLeft.value = !isAllowLeft.value
}
const allowRightCategory = () => {
  isAllowRight.value = !isAllowRight.value
}

const getData = async () => {
  try {
    const res = await getStorage()
    if (res?.data) {
      data.value = JSON.parse(res?.data)
    }
  } catch (error) {
    console.log('error', error);
  }
}

onMounted(() => {
  getData()
})

function generateUniqueId() {
  const timestamp = Date.now().toString(36); // 将时间戳转换为 36 进制字符串
  const random = Math.random().toString(36).substring(2, 8); // 生成随机字符串
  return timestamp + random; // 组合时间戳和随机数
}

const setStorage = (data: any) => {
  return uni.setStorage({
    key: 'formData',
    data,
    // success: function () {
    //   console.log('setData-success');
    // }
  });
}
const getStorage = () => {
  return uni.getStorage({
    key: 'formData',
    // success: function (res) {
    //   console.log(res.data);
    // }
  });
}
const removeStorage = () => {
  return uni.removeStorage({
    key: 'formData',
    // success: function (res) {
    //   console.log('success');
    // }
  });
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
.common-input {
  border: 1px solid #eee;
  border-radius: 2px;
  padding: 2px 5px;
  margin: 5px 0px 10px 0px;
}
.common-empty-text {
  color: var(--primary-color);
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
    .all-search {
      margin-right: 5px;
    }
  }
  .search-btn {
    display: flex;
    align-items: center;
    width: calc(100% - 10px);
    height: 40px;
    margin: 0 0 5px 5px;
    .search-btn-right {
      display: flex;
      align-items: center;
      height: 100%;
      margin-left: auto;
      .btn-line {
        height: 100%;
        width: 2px;
        background: var(--primary-color);
        margin: 0 5px;
      }
    }
  }
  .content {
    display: flex;
    width: 100%;
    height: calc(100% - 50px - 50px);
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
        .category-item-active {
          background: var(--primary-color);
          .name {
            color: #fff;
          }
        }
      }
      .category-allow {
        position: absolute;
        left: 5px;
        bottom: 12px;
        width: 20px;
        height: 20px;
        background: var(--primary-color);
        z-index: 10;
        &::after, &::before {
          content: '';
          position: absolute;
          left: 0;
          top: 6px;
          width: 100%;
          height: 2px;
          background: var(--text-color);
        }
        &::before {
          top: 12px;
        }
      }
      .add-category {
        position: absolute;
        right: 5px;
        bottom: 5px;
      }
    }
    .allow-left {
      width: 25%;
    }
    .allow-right-left {
      width: calc(50%);
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
      .category-allow-right {
        position: absolute;
        left: 5px;
        bottom: 12px;
        width: 20px;
        height: 20px;
        background: var(--primary-color);
        z-index: 10;
        &::after, &::before {
          content: '';
          position: absolute;
          left: 0;
          top: 6px;
          width: 100%;
          height: 2px;
          background: var(--text-color);
        }
        &::before {
          top: 12px;
        }
      }
      .add-commodity {
        position: absolute;
        right: 5px;
        bottom: 5px;
      }
    }
    .allow-right {
      width: calc(75% - 2px);
    }
    .allow-right-right {
      width: calc(50% - 2px);
    }
  }
}
</style>
