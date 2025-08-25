## 描述
分为左中右三个部分，左侧选择需要使用的组件，中间实时预览效果，右侧编辑组件具体配置

## 环境和技术栈
web vue2 antdv1.7.8

## 效果
![[Snipaste_2025-08-22_18-23-02.png]]

## 思路
![[Snipaste_2025-08-25_09-47-49.png]]

## 实现

**template部分**
```html
<template>
  <a-spin :spinning="confirmLoading">
    <div class="activity-config-container">
      <div class="top">
        <a-button @click="back">退出编辑器</a-button>
        <a-button type="primary" @click="confirm">提交发布</a-button>
      </div>
      <div class="left">
        <h3>组件</h3>
        <div class="comp-list">
          <div class="comp-item" @click.stop="addMod('image')">
            <div>图片</div>
            <a-icon :component="ImageSvg" style="font-size: 45px;" />
          </div>
          <div class="comp-item" @click.stop="addMod('coupon')">
            <div>优惠券</div>
            <a-icon :component="CouponSvg" style="font-size: 45px;" />
          </div>
          <div class="comp-item" @click.stop="addMod('grouping')">
            <div>商品组</div>
            <a-icon :component="ProductGroupSvg" style="font-size: 45px;" />
          </div>
        </div>
      </div>
      <div class="middle">
        <h3>预览</h3>
        <div class="phone-box">
          <img class="topBg" src="@/assets/mpbg.png" alt="">
          <h4>{{ activity.name }}</h4>
          <div ref="modContainer" class="container">
            <div v-for="(mod, idx) in mods" :key="mod.id" class="mod-wrapper" @click="editMod(mod)" :class="{'active': currMod === mod}">
              <div class="inner-capsule">
                <a-icon type="arrow-up" @click="onInnerMod('up', mod)" />
                <a-icon type="arrow-down" @click="onInnerMod('down', mod)" />
                <a-icon type="delete" @click="onInnerMod('delete', mod)" />
              </div>
              <template v-if="mod.type === 'image'" >
                <div class="image-preview-wrapper">
                  <img v-if="mod.image_img" :src="mod.image_img" alt="" >
                  <div v-else class="empty">请点击完善图片配置</div>
                </div>
              </template>
              <template v-if="mod.type === 'coupon'" >
                <div class="coupon-preview-wrapper">
                  <img v-if="mod.coupon_noReceiveImg" :src="mod.coupon_noReceiveImg" alt="" >
                  <div v-else class="empty">请点击完善优惠券配置</div>
                </div>
              </template>
              <template v-if="mod.type === 'grouping'" >
                <div
                  class="product-group-preview-wrapper"
                  :style="{
                    '--look-more-colour': mod.grouping_lookMoreColour,
                    '--background': mod.grouping_background,
                    '--tag-colour': mod.grouping_tagColour,
                    '--tag-background': mod.grouping_tagBackground,
                  }"
                >
                  <template v-if="mod.tempProducts.length > 0">
                    <div class="product-list">
                      <div v-for="(product, idx) in mod.tempProducts" class="product">
                        <div class="product_tag" v-if="mod.grouping_tag[idx].name">
                          {{ mod.grouping_tag[idx].name }}
                        </div>
                        <div class="product_img">
                          <img :src="product.publicImg" alt="">
                        </div>
                        <div class="product_name">{{ product.cnName + product.seriesName + product.modelName }}</div>
                        <div class="product_price" v-if="mod.grouping_priceType === 'grouping_dayPrice'">
                          <span>￥</span>
                          <span>{{ product.dayPrice }}</span>
                          <span>/日</span>
                        </div>
                        <div class="product_price" v-else>
                          <span>￥</span>
                          <span>{{ product.salePrice }}</span>
                        </div>
                      </div>
                    </div>
                    <div class="more">查看更多&gt;</div>
                  </template>
                  <div class="empty" v-else>请点击完善商品组配置</div>
                </div>
              </template>
              
            </div>
          </div>
        </div>
      </div>
      <div class="right">
        <h3>配置</h3>
        <div>
          <template v-if="model.type === 'image'">
            <a-form-model ref="form" v-bind="layout" :model="model" :rules="typeImageRules">
              <a-form-model-item label="图片" prop="image_img">
                <JImageUpload class="avatar-uploader" text="上传" v-model="model.image_img" :fileType="2" :isCompressor="false" />
              </a-form-model-item>
              <a-form-model-item label="跳转地址" prop="image_skipAddress">
                <a-radio-group v-model="model.image_skipType" @change="model.image_skipAddress = ''">
                  <a-radio value="image_noSkip">无跳转</a-radio>
                  <a-radio value="image_aliPaySkip">alipay链接</a-radio>
                  <a-radio value="image_h5Skip">H5链接</a-radio>
                </a-radio-group>
                <a-input 
                  v-if="model.image_skipType!='image_noSkip'" 
                  v-model="model.image_skipAddress" 
                  placeholder="请输入跳转地址" 
                  @change="() => {$refs.form.validateField(['image_skipAddress'])}"
                />
              </a-form-model-item>
            </a-form-model>
          </template>
          <template v-else-if="model.type === 'coupon'">
            <a-form-model ref="form" v-bind="layout" :model="model" :rules="typeCouponRules">
              <a-form-model-item label="未领取" prop="coupon_noReceiveImg">
                <JImageUpload class="avatar-uploader" text="上传" v-model="model.coupon_noReceiveImg" :fileType="2" :isCompressor="false" />
              </a-form-model-item>
              <a-form-model-item label="已领取" prop="coupon_yesReceiveImg">
                <JImageUpload class="avatar-uploader" text="上传" v-model="model.coupon_yesReceiveImg" :fileType="2" :isCompressor="false" />
              </a-form-model-item>
              <a-form-model-item label="优惠券" prop="coupon_coupon">
                <a-button type="primary" @click="onSelectCoupon">选择优惠券（{{model.coupon_coupon.length}}/10）</a-button>
                <div class="coupon-list">
                  <a-tag v-for="item in model.coupon_coupon" :key="item.id" closable @close="onDeleteCoupon(item)" color="pink">
                    {{ item.name }}
                  </a-tag>
                </div>
              </a-form-model-item>
            </a-form-model>
          </template>
          <template v-else-if="model.type === 'grouping'">
            <a-form-model ref="form" v-bind="layout" :model="model" :rules="typeProductGroupRules">
              <a-form-model-item label="商品组" prop="groupId">
                <a-button type="primary" @click="onSelectProductGroup">选择商品组（{{model.groupId ? 1 : 0}}/1）</a-button>
                <div class="product-group-list">
                  <a-tag v-if="model.groupId" closable @close="onDeleteProductGroup" color="blue">
                    {{ model.groupName }}
                  </a-tag>
                </div>
              </a-form-model-item>
              <a-form-model-item label="商品数量" prop="grouping_num">
                <a-input v-model="model.grouping_num" placeholder="请输入商品数量" @input="onInputCount" />
              </a-form-model-item>
              <a-form-model-item label="价格类型" prop="grouping_priceType">
                <a-radio-group v-model="model.grouping_priceType">
                  <a-radio value="grouping_dayPrice">日租金</a-radio>
                  <a-radio value="grouping_salePrice">销售价</a-radio>
                </a-radio-group>
              </a-form-model-item>
              <a-form-model-item label="查看更多" prop="grouping_lookMoreColour">
                <input class="color-picker" v-model="model.grouping_lookMoreColour" placeholder="请输入更多颜色" v-huebee />
              </a-form-model-item>
              <a-form-model-item label="背景色值" prop="grouping_background">
                <input class="color-picker" v-model="model.grouping_background" placeholder="请输入背景颜色" v-huebee/>
              </a-form-model-item>
              <a-divider />
              <a-form-model-item label="商品标签" prop="grouping_tag">
                <div class="tag-list">
                  <template v-for="(item, idx) in model.grouping_tag">
                    <a-input v-if="idx < Number(model.grouping_num)" :key="item.id" v-model="item.name" />
                  </template>
                </div>
              </a-form-model-item>
              <a-form-model-item label="标签字色" prop="grouping_tagColour">
                <input class="color-picker" v-model="model.grouping_tagColour" placeholder="请输入标签字色" v-huebee />
              </a-form-model-item>
              <a-form-model-item label="标签背景" prop="grouping_tagBackground">
                <input class="color-picker" v-model="model.grouping_tagBackground" placeholder="请输入标签背景" v-huebee />
              </a-form-model-item>
            </a-form-model>
          </template>
        </div>
      <SelectProductGroup ref="selectProductGroup" @refresh="onSelectProductsRefresh"></SelectProductGroup>
      <SelectCoupon ref="selectCoupon" @refresh="onSelectCouponRefresh"></SelectCoupon>
      </div>

    </div>
  </a-spin>
</template>
```

**js部分**
```js

<script>
import CouponSvg from './components/couponSvg.vue'
import ImageSvg from './components/imageSvg.vue'
import ProductGroupSvg from './components/productGroupSvg.vue'
import { numSanitize } from '@/utils/numSanitize'
import { cloneDeep } from 'lodash'
import SelectProductGroup from './components/selectProductGroup.vue'
import SelectCoupon from './components/selectCoupon.vue'
import { getAction, postAction } from '@api/manage'
export default {
  name: 'ActivityConfig',
  components: {
    SelectProductGroup,
    SelectCoupon
  },
  data() {
    return {
      vars: {
        product_img_width: 96,
        product_img_height: 96,
      },
      layout: {
        labelCol: { span: 5 },
        wrapperCol: { span: 14 }
      },
      activity: {
        name: '测试活动名称'
      },
      CouponSvg,
      ImageSvg,
      ProductGroupSvg,
      confirmLoading: false,
      currMod: null,
      
      mods: [],
      model: {},
      typeImageRules: {
        image_img: [
          { required: true, message: '请上传图片!' }
        ],
        image_skipAddress: [
          { 
            validator: (rule, value, callback) => {
              if((this.model.image_skipType !='image_noSkip') && !value){
                callback(new Error('请输入跳转地址!'))
              }else{
                callback()
              }
            }
          }
        ],
      },
      typeCouponRules: {
        coupon_noReceiveImg: [
          { required: true, message: '请上传未领取图片!' }
        ],
        coupon_yesReceiveImg: [
          { required: true, message: '请上传已领取图片!' }
        ],
        coupon_coupon: [
          { required: true, message: '请选择优惠券!' },
          { 
            validator: (rule, value, callback) => {
              if(value && value.length === 0){
                callback(new Error('至少选择1个优惠券!'))
              }else if(value && value.length > 10){
                callback(new Error('最多选择10个优惠券!'))
              }else{
                callback()
              }
            } 
          }
        ]
      },
      typeProductGroupRules: {
        groupId: [
          { required: true, message: '请选择商品组!' }
        ],
        grouping_num: [
          { required: true, message: '请输入商品数量!' },
          {
            validator: (rule, value, callback) => {
              if(Number(value) < 1 || Number(value) > 9){
                callback(new Error('商品数量范围为1-9'))
              }else{
                callback()
              }
            }
          }
        ],
        grouping_priceType: [
          { required: true, message: '请选择价格类型!' }
        ],
        grouping_tag: [
          { 
            validator: (rule, value, callback) => {
              const isError = value.some(i => {
                return i.name.length > 6
              })
              if(isError){
                callback(new Error('商品标签长度不能超过6个字符!'))
              }else{
                callback()
              }
            },
          }
        ],
      },
    }
  },
  created(){
    this.getActivityInfo()
  },
  methods: {
    getActivityInfo(){
      this.confirmLoading = true
      postAction('/opActivityType/getActivityType', {
        activityId: this.$route.query.id
      }).then(async (res) => {
        this.confirmLoading = false
        if(res.success){
          this.activity = {
            activityId: res.result.activityId,
            activityName: res.result.activityName,
          }
          if(res.result.typeValueList){
            const typeValueList = res.result.typeValueList.map((item) => {
              const newItem = {
                id: item.id,
                type: item.type,
              }
              if(item.type === 'image'){
                newItem.image_img = item.valueJson.image_img
                newItem.image_skipType = item.valueJson.image_skipType
                newItem.image_skipAddress = item.valueJson.image_skipAddress
              }else if(item.type === 'coupon'){
                newItem.coupon_noReceiveImg = item.valueJson.coupon_noReceiveImg
                newItem.coupon_yesReceiveImg = item.valueJson.coupon_yesReceiveImg
                newItem.coupon_coupon = JSON.parse(item.valueJson.coupon_coupon)
              }else if(item.type === 'grouping'){
                newItem.groupId = JSON.parse(item.valueJson.grouping_grouping).id
                newItem.groupName = JSON.parse(item.valueJson.grouping_grouping).name
                newItem.grouping_num = item.valueJson.grouping_num
                newItem.grouping_priceType = item.valueJson.grouping_priceType
                // 回显标签时，按照商品数量取标签列表中的元素，其余补全9个为空字符串
                newItem.grouping_tag = []
                const srcGroupingTag = JSON.parse(item.valueJson.grouping_tag).map((i, idx) => ({id: idx, name: i}))
                for(let i = 0; i < 9; i++){
                  if(i < Number(newItem.grouping_num)){
                    newItem.grouping_tag.push(srcGroupingTag[i] || {id: Math.random().toString(36).slice(2), name: ''})
                  }else{
                    newItem.grouping_tag.push({id: Math.random().toString(36).slice(2), name: ''})
                  }
                }
                newItem.grouping_tagColour = item.valueJson.grouping_tagColour || '#ffffff'
                newItem.grouping_tagBackground = item.valueJson.grouping_tagBackground || '#ff6f00'
                newItem.grouping_lookMoreColour = item.valueJson.grouping_lookMoreColour || '#000000'
                newItem.grouping_background = item.valueJson.grouping_background || '#ffffff'
                newItem.products = []
                newItem.tempProducts = this.getTempProducts(newItem.products, newItem.grouping_num)
                this.getGroupProduct(newItem.groupId).then((products) => {
                  newItem.products = products || []
                  newItem.tempProducts = this.getTempProducts(newItem.products, newItem.grouping_num)
                })
              }
              return newItem
            })
            this.mods = typeValueList
            console.log('get', this.mods);
          }else{
            this.mods = []
          }
        }else{
          this.$message.error(res.msg ? res.msg : res.message)
        }
      })
    },
    test(){
      console.log(this.model);
    },
    back(noConfirm = false) {
      if(noConfirm) {
        this.$bus.$emit('TabRemove', {
          key: this.$route.fullPath,
          disableAutoActive: true
        })
        this.$router.back()
        return
      }
      this.$confirm({
        title: '退出编辑器',
        content: '未保存内容将丢失，确认返回吗？',
        onOk: () => {
          this.$bus.$emit('TabRemove', {
            key: this.$route.fullPath,
            disableAutoActive: true
          })
          this.$router.back()
        }
      })
    },
    validate(){
      if(!this.mods.length) return this.$message.error('请添加模块!')
      let errorInfo = ''
      for(let i = 0; i < this.mods.length; i++){
        const mod = this.mods[i]
        const type = mod.type
        const modIdx = i + 1
        if(type === 'image'){
          if(!mod.image_img){
            errorInfo = `第${modIdx}个模块【图片】：请上传图片!`
          }else if(mod.image_skipType !== 'image_noSkip' && !mod.image_skipAddress){
            errorInfo = `第${modIdx}个模块【图片】：请输入跳转地址!`
          }
        }else if(type === 'coupon'){
          if(!mod.coupon_noReceiveImg){
            errorInfo = `第${modIdx}个模块【优惠券】：请上传未领取图片!`
          }else if(!mod.coupon_yesReceiveImg){
            errorInfo = `第${modIdx}个模块【优惠券】：请上传已领取图片!`
          }else if(!Array.isArray(mod.coupon_coupon)){
            errorInfo = `第${modIdx}个模块【优惠券】：请选择优惠券!`
          }else if(mod.coupon_coupon.length === 0){
            errorInfo = `第${modIdx}个模块【优惠券】：至少选择1个优惠券!`
          }else if(mod.coupon_coupon.length > 10){
            errorInfo = `第${modIdx}个模块【优惠券】：最多选择10个优惠券!`
          }
        }else if(type === 'grouping'){
          if(!mod.groupId){
            errorInfo = `第${modIdx}个模块【商品组】：请选择商品组!`
          }else if(mod.grouping_num === '' || mod.grouping_num === null || mod.grouping_num === undefined){
            errorInfo = `第${modIdx}个模块【商品组】：请输入商品数量!`
          }else if(Number(mod.grouping_num) < 1 || Number(mod.grouping_num) > 9){
            errorInfo = `第${modIdx}个模块【商品组】：商品数量范围为1-9`
          }else if(!mod.grouping_priceType){
            errorInfo = `第${modIdx}个模块【商品组】：请选择价格类型!`
          }else if(Array.isArray(mod.grouping_tag)){
            const hasTooLong = mod.grouping_tag.some(t => (t && t.name ? String(t.name).length : 0) > 6)
            if(hasTooLong){
              errorInfo = `第${modIdx}个模块【商品组】：商品标签长度不能超过6个字符!`
            }
          }
        }
        if(errorInfo){
          this.currMod = mod
          this.model = mod
          this.$message.error(errorInfo)
          this.$nextTick(() => {
            const wrappers = this.$refs.modContainer ? this.$refs.modContainer.querySelectorAll('.mod-wrapper') : []
            const errorEl = wrappers[i]
            if(errorEl && errorEl.scrollIntoView){
              errorEl.scrollIntoView({ behavior: 'smooth', block: 'center' })
            }
            this.$refs.form.validate()
          })
          break
        }
      }
      return errorInfo
    },
    confirm() {
      // 由于表单校验只能校验当前编辑的模块，所以需要对全部模块进行手动校验
      const errorInfo = this.validate()
      if(errorInfo) return

      this.$confirm({
        title: '提交发布',
        content: '确认提交发布吗？',
        onOk: () => {
          const typeValueList = this.mods.map((item, index) => {
            const newItem = {
              type: item.type,
              sort: index + 1,
              valueJson: {}
            }
            if(item.type === 'image'){
              newItem.valueJson = {
                image_img: item.image_img,
                image_skipType: item.image_skipType,
                image_skipAddress: item.image_skipType === 'image_noSkip' ? '' : item.image_skipAddress,
              }
            }else if(item.type === 'coupon'){
              newItem.valueJson = {
                coupon_noReceiveImg: item.coupon_noReceiveImg,
                coupon_yesReceiveImg: item.coupon_yesReceiveImg,
                coupon_coupon: JSON.stringify(item.coupon_coupon),
              }
            }else if(item.type === 'grouping'){
              // 提交标签时，按照商品数量取标签列表中的元素，其余补全9个为空字符串
              const grouping_tag = []
              for(let i = 0; i < 9; i++){
                if(i < Number(item.grouping_num)){
                  grouping_tag.push(item.grouping_tag[i].name || '')
                }else{
                  grouping_tag.push('')
                }
              }
              newItem.valueJson = {
                grouping_grouping: JSON.stringify({id: item.groupId, name: item.groupName}),
                grouping_num: item.grouping_num,
                grouping_priceType: item.grouping_priceType,
                grouping_tag: JSON.stringify(grouping_tag),
                grouping_tagColour: item.grouping_tagColour,
                grouping_background: item.grouping_background,
                grouping_tagBackground: item.grouping_tagBackground,
                grouping_lookMoreColour: item.grouping_lookMoreColour,
              }
            }
            return newItem
          })
          this.confirmLoading = true
          postAction('/opActivityType/saveOrUpdateActivityType', {
            activityId: this.activity.activityId,
            typeValueList
          }).then(res => {
            this.confirmLoading = false
            if(res.success){
              this.$message.success('提交发布成功')
              setTimeout(() => {
                this.back(true)
              }, 200)
            }else{
              this.$message.error(res.msg ? res.msg : res.message)
            }
          })
        }
      })
    },
    addMod(type) {
      const newMod = {
        id: Math.random().toString(36).substring(2, 15),
        type,
      }
      if(type === 'image'){
        newMod.image_img = ''
        newMod.image_skipType = 'image_noSkip'
        newMod.image_skipAddress = ''
      }else if(type === 'coupon'){
        newMod.coupon_yesReceiveImg = ''
        newMod.coupon_noReceiveImg = ''
        newMod.coupon_coupon = []
      }else if(type === 'grouping'){
        newMod.groupId = ''
        newMod.groupName = ''
        newMod.grouping_num = ''
        newMod.grouping_priceType = 'grouping_dayPrice'
        newMod.grouping_lookMoreColour = '#000000'
        newMod.grouping_background = '#ffffff'
        newMod.grouping_tag = Array.from({length: 9}, () => ({id: Math.random().toString(36).slice(2), name: ''}))
        newMod.grouping_tagColour = '#ffffff'
        newMod.grouping_tagBackground = '#ff6f00'
        newMod.products = []
        newMod.tempProducts = this.getTempProducts(newMod.products, newMod.grouping_num)
      }
      this.mods.push(newMod)
      this.currMod = newMod
      this.model = newMod
    },
    editMod(item) {
      this.currMod = item
      this.model = item
      console.log('edit', item);
    },
    onInnerMod(type, targetMod){
      const modIdx = this.mods.findIndex(mod => mod.id === targetMod.id)
      if(modIdx === -1) return
      if(type === 'up'){
        if(modIdx > 0){
          const item = this.mods.splice(modIdx, 1)[0]
          this.mods.splice(modIdx - 1, 0, item)
        }
      }else if(type === 'down'){
        if(modIdx < this.mods.length - 1){
          const item = this.mods.splice(modIdx, 1)[0]
          this.mods.splice(modIdx + 1, 0, item)
        }
      }else if(type === 'delete'){
        this.mods = this.mods.filter(mod => mod.id !== targetMod.id)
      }
    },
    /* 优惠券 */
    onDeleteCoupon(item) {
      this.model.coupon_coupon = this.model.coupon_coupon.filter(coupon => coupon.id !== item.id)
    },
    onSelectCoupon() {
      this.$refs.selectCoupon.show()
    },
    onSelectCouponRefresh(data){
      data.forEach(item => {
        if(this.model.coupon_coupon.some(jtem => jtem.id === item.id)) return
        this.model.coupon_coupon.push({
          id: item.id,
          name: item.name,
        })
      })
      this.$nextTick(() => {
        this.$refs.form.validateField('coupon_coupon')
      })
    },
    /* 商品组 */
    getGroupProduct(groupId){
      this.confirmLoading = true
      return postAction('/proGrouping/listProGroupingProductLimit9', {
        groupingId: groupId,
        pageNumber: 1,
        pageSize: 9
      }).then(res => {
        this.confirmLoading = false
        if(res.success){
          return res.result
        }else{
          this.$message.error(res.msg ? res.msg : res.message)
          return null
        }
      })
    },
    getTempProducts(srcProducts, num){
      if(!srcProducts || !Array.isArray(srcProducts)) return []
      if(Array.isArray(srcProducts) && !srcProducts.length)return []
      const rNum = Number(num)
      return rNum ? srcProducts.slice(0, rNum) : []
    },
    onSelectProductGroup() {
      this.$refs.selectProductGroup.show()
    },
    onDeleteProductGroup() {
      this.model.groupId = ''
      this.model.groupName = ''
      this.model.products = []
      this.model.tempProducts = this.getTempProducts(this.model.products, this.model.grouping_num)
    },
    onSelectProductsRefresh(data){
      data.forEach(item => {
        this.model.groupId = item.id
        this.model.groupName = item.name
      })
      this.getGroupProduct(this.model.groupId).then(products => {
        this.model.products = products || []
        this.model.tempProducts = this.getTempProducts(this.model.products, this.model.grouping_num)
      })
      this.$nextTick(() => {
        this.$refs.form.validateField('groupId')
      })
    },
    onInputCount(){
      this.model.grouping_num = numSanitize(this.model.grouping_num)
      this.model.tempProducts = this.getTempProducts(this.model.products, this.model.grouping_num)
    }
  }
}
</script>
```
**样式部分**
```css
<style lang="less" scoped>
.activity-config-container {
  height: 100%;
  width: 100%;
  box-sizing: border-box;
  padding: 12px;
  display: grid;
  grid-template-columns: minmax(360px, 560px) 375px minmax(480px, 1fr);
  // grid-template-rows: auto 676px;
  grid-template-areas:
    'top top top'
    'left middle right';
  gap: 15px;
}

.top {
  grid-area: top;
  min-height: 56px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 12px;
  background: #f5f5f5;
  border: 1px solid #e8e8e8;
  border-radius: 4px;
}

.left { grid-area: left; }
.middle { grid-area: middle; }
.right { grid-area: right; }

h3{
  font-size: 20px;
  font-weight: bold;
  text-align: center;
}

.left{
  .comp-list{
    display: grid;
    grid-template-columns: repeat(auto-fit, 108px);
    gap: 12px;
    .comp-item{
      width: 108px;
      height: 108px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      background: #ffffff;
      border: 1px solid #e8e8e8;
      border-radius: 8px;
      cursor: pointer;
      transition: all .2s ease;
      &:hover{
        border-color: #1890ff;
        transform: translateY(-2px);
      }
      >div{
        font-size: 12px;
        color: #666666;
        margin-bottom: 6px;
      }
    }
  }
}

.middle{
  position: relative;
  .phone-box{
    width: 375px;
    height: 676px;
    border: 1px solid #e8e8e8;
    border-radius:15px;
    background: #f5f5f5;
    position: relative;
    overflow-y: auto;
    .topBg{
      width: 100%;
      display: block;
    }
    h4{
      font-size: 14px;
      margin: 0;
      font-weight: bold;
      position: absolute;
      top: 55px;
      left: 50%;
      transform: translateX(-50%);
      color: #000000;
    }
    .empty{
      text-align: center;
      color: #999999;
      font-size: 12px;
      height: 100px;
      background: #fff;
      line-height: 100px;
    }
    .mod-wrapper:has(.empty){
      margin-top: 5px;
      margin-bottom: 5px;
      border: 1px solid red !important;
    }
    .container{
      width: 100%;
      display: flex;
      flex-direction: column;
      .mod-wrapper{
        position: relative;
        &.active{
          border: 1px solid #1890ff;
        }
        .inner-capsule{
          position: absolute;
          top: 4px;
          left: 4px;
          display: none;
          flex-direction: column;
          background: #FFFFFF;
          border-radius: 4px;
          z-index: 1;
          box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.2);
          overflow: hidden;
          >*{
            padding: 5px 4px;
            &:hover{
              background: rgba(0, 0, 0, 0.1);
            }
          }
        }
        &:hover .inner-capsule{
          display: flex;
        }
        .image-preview-wrapper, .coupon-preview-wrapper{
          img{
            width: 100%;
          }
        }
        .product-group-preview-wrapper{
          background: var(--background);
          .product-list{
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 4px;
            padding: 0 12px;
            >div{
              position: relative;
              background: #ffffff;
              .product_tag{
                position: absolute;
                top: 0;
                left: 0;
                background: var(--tag-background);
                color: var(--tag-colour);
                border-radius: 0px 0px 8px 0px;
                font-size: 10px;
                line-height: 10px;
                padding: 3px 4px;
              }
              .product_img{
                width: 100%;
                height: 96px;
                img{
                  width: 100%;
                  height: 100%;
                  object-fit: contain;
                }
              }
              .product_name{
                font-size: 12px;
                line-height: 17px;
                color: #1E2234;
                margin-top: 8px;
                overflow: hidden;
                text-overflow: ellipsis;
                display: -webkit-box;
                -webkit-line-clamp: 2;
                -webkit-box-orient: vertical;
                word-break: break-all;
                text-align: center;
                padding: 0 8px;
              }
              .product_price{
                text-align: center;
                font-size: 10px;
                color: #1E2234;
                font-weight: medium;
                line-height: 29px;
                span:nth-child(2){
                  font-size: 12px;
                }
              }
            }
          }
          .more{
            padding: 16px 0;
            text-align: center;
            font-size: 12px;
            color: var(--look-more-colour);
          }
        }
      }
    }
  }
}

.right{
  >div{
    background: #ffffff;
    padding: 20px 0;
    min-height: 676px;
    border-radius: 15px;
    border: 1px solid #e8e8e8;
  }
  .color-picker{
    line-height: 21px;
    padding: 4px 11px;
    font-size: 14px;
    border: 1px solid #d9d9d9;
    border-radius: 4px;
  }
  .tag-list{
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 5px;
  }
}
</style>
```