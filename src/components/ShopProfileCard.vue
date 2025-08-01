<template>
  <v-card class="elevation-3" rounded="lg">
    <v-card-title class="bg-primary text-white pa-4">
      <v-icon left class="mr-2">mdi-store</v-icon>
      店铺配置
    </v-card-title>
    
    <v-card-text class="pa-0">
      <v-list>
        <!-- 当前店铺 -->
        <v-list-item class="pa-4">
          <template v-slot:prepend>
            <v-avatar color="primary" class="mr-4">
              <v-icon color="white">mdi-store-outline</v-icon>
            </v-avatar>
          </template>
          
          <v-list-item-title class="text-h6 mb-2">当前店铺</v-list-item-title>
          <v-list-item-subtitle class="mb-3">
            {{ currentShop.name }} - {{ currentShop.id }}
          </v-list-item-subtitle>
          
          <template v-slot:append>
            <v-btn 
              color="primary" 
              variant="outlined"
              size="small"
              @click="openShopDialog"
            >
              切换店铺
            </v-btn>
          </template>
        </v-list-item>

        <v-divider></v-divider>

        <!-- 收货地址 -->
        <v-list-item class="pa-4">
          <template v-slot:prepend>
            <v-avatar color="success" class="mr-4">
              <v-icon color="white">mdi-map-marker</v-icon>
            </v-avatar>
          </template>
          
          <v-list-item-title class="text-h6 mb-3">收货地址</v-list-item-title>
          
          <v-radio-group v-model="addressType" class="mt-0">
            <v-radio 
              label="使用当前店铺地址" 
              value="shop"
              color="success"
            ></v-radio>
            <v-radio 
              label="自定义收货地址" 
              value="custom"
              color="success"
            ></v-radio>
          </v-radio-group>
          
          <v-expand-transition>
            <div v-if="addressType === 'shop'" class="mt-2">
              <v-card variant="outlined" class="pa-3 bg-grey-lighten-5">
                <div class="text-body-2">
                  <div><strong>{{ currentShop.name }}</strong></div>
                  <div>{{ currentShop.address }}</div>
                  <div>联系人：{{ currentShop.contact }}</div>
                  <div>电话：{{ currentShop.phone }}</div>
                </div>
              </v-card>
            </div>
          </v-expand-transition>
          
          <v-expand-transition>
            <div v-if="addressType === 'custom'" class="mt-3">
              <v-text-field
                v-model="customAddress.name"
                label="收货人姓名"
                variant="outlined"
                density="compact"
                class="mb-2"
              ></v-text-field>
              <v-text-field
                v-model="customAddress.phone"
                label="联系电话"
                variant="outlined"
                density="compact"
                class="mb-2"
              ></v-text-field>
              <v-textarea
                v-model="customAddress.address"
                label="详细地址"
                variant="outlined"
                density="compact"
                rows="2"
              ></v-textarea>
            </div>
          </v-expand-transition>
        </v-list-item>

        <v-divider></v-divider>

        <!-- 电子邮箱 -->
        <v-list-item class="pa-4">
          <template v-slot:prepend>
            <v-avatar color="info" class="mr-4">
              <v-icon color="white">mdi-email</v-icon>
            </v-avatar>
          </template>
          
          <v-list-item-title class="text-h6 mb-2">电子邮箱</v-list-item-title>
          
          <div v-if="!email.bound" class="d-flex flex-column">
            <v-list-item-subtitle class="mb-3 text-grey-darken-1">
              未绑定邮箱
            </v-list-item-subtitle>
            <v-text-field
              v-model="email.input"
              label="输入邮箱地址"
              variant="outlined"
              density="compact"
              type="email"
              :rules="emailRules"
              class="mb-2"
            ></v-text-field>
            <v-btn 
              color="info" 
              variant="flat"
              size="small"
              @click="bindEmail"
              :disabled="!isEmailValid"
            >
              绑定邮箱
            </v-btn>
          </div>
          
          <div v-else class="d-flex flex-column">
            <div class="d-flex align-center mb-3">
              <v-chip color="success" variant="flat" size="small" class="mr-2">
                <v-icon left size="small">mdi-check-circle</v-icon>
                已绑定
              </v-chip>
              <span class="text-body-2">{{ email.address }}</span>
            </div>
            <v-btn 
              color="error" 
              variant="outlined"
              size="small"
              @click="unbindEmail"
            >
              解绑邮箱
            </v-btn>
          </div>
        </v-list-item>
      </v-list>
    </v-card-text>

    <!-- 店铺切换对话框 -->
    <v-dialog v-model="shopDialog" max-width="500px">
      <v-card>
        <v-card-title class="bg-primary text-white">
          <v-icon left class="mr-2">mdi-store-search</v-icon>
          选择店铺
        </v-card-title>
        
        <v-card-text class="pa-0">
          <v-list>
            <v-list-item
              v-for="shop in availableShops"
              :key="shop.id"
              @click="selectShop(shop)"
              :class="{ 'bg-grey-lighten-4': shop.id === currentShop.id }"
            >
              <template v-slot:prepend>
                <v-avatar color="primary" size="small">
                  <v-icon color="white" size="small">mdi-store</v-icon>
                </v-avatar>
              </template>
              
              <v-list-item-title>{{ shop.name }}</v-list-item-title>
              <v-list-item-subtitle>{{ shop.address }}</v-list-item-subtitle>
              
              <template v-slot:append v-if="shop.id === currentShop.id">
                <v-chip color="success" size="small">当前</v-chip>
              </template>
            </v-list-item>
          </v-list>
        </v-card-text>
        
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="shopDialog = false">取消</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

interface Shop {
  id: string
  name: string
  address: string
  contact: string
  phone: string
}

interface CustomAddress {
  name: string
  phone: string
  address: string
}

interface Email {
  bound: boolean
  address: string
  input: string
}

// 响应式数据
const shopDialog = ref(false)
const addressType = ref('shop')
const currentShop = ref<Shop>({
  id: 'SHOP001',
  name: '总店',
  address: '北京市朝阳区某某街道123号',
  contact: '张经理',
  phone: '138-0000-0000'
})

const customAddress = ref<CustomAddress>({
  name: '',
  phone: '',
  address: ''
})

const email = ref<Email>({
  bound: false,
  address: '',
  input: ''
})

const availableShops = ref<Shop[]>([
  {
    id: 'SHOP001',
    name: '总店',
    address: '北京市朝阳区某某街道123号',
    contact: '张经理',
    phone: '138-0000-0000'
  },
  {
    id: 'SHOP002',
    name: '分店-西城店',
    address: '北京市西城区某某大厦456号',
    contact: '李经理',
    phone: '138-1111-1111'
  },
  {
    id: 'SHOP003',
    name: '分店-海淀店',
    address: '北京市海淀区某某科技园789号',
    contact: '王经理',
    phone: '138-2222-2222'
  }
])

// 邮箱验证规则
const emailRules = [
  (v: string) => !!v || '请输入邮箱地址',
  (v: string) => /.+@.+\..+/.test(v) || '请输入有效的邮箱地址',
]

// 计算属性
const isEmailValid = computed(() => {
  return email.value.input && /.+@.+\..+/.test(email.value.input)
})

// 方法
const openShopDialog = () => {
  shopDialog.value = true
}

const selectShop = (shop: Shop) => {
  currentShop.value = shop
  shopDialog.value = false
}

const bindEmail = () => {
  if (isEmailValid.value) {
    email.value.bound = true
    email.value.address = email.value.input
    email.value.input = ''
  }
}

const unbindEmail = () => {
  email.value.bound = false
  email.value.address = ''
  email.value.input = ''
}
</script>

<style scoped>
.v-list-item {
  min-height: auto;
}

.v-card-title {
  font-weight: 500;
}
</style>