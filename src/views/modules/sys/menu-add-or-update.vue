<template>
  <el-dialog :title="!dataForm.id ? $t('sysManagement.add') : $t('sysManagement.modify')" :close-on-click-modal="false"
    :before-close="beforeClose" :visible.sync="visible">
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm" size="small"
      @keyup.enter.native="dataFormSubmit()" label-width="155px">
      <el-form-item :label="$t('sysManagement.type')" prop="type">
        <el-radio-group v-model="dataForm.type" @change="() => { dataForm.type === 0 ? dataForm.hidden = 0 : '' }">
          <el-radio v-for="(type, index) in dataForm.typeList" :label="index" :key="index">{{ type }}</el-radio>
        </el-radio-group>
      </el-form-item>
      <!-- 语言选择框 -->
      <el-form-item :label="this.$i18n.t('product.selectLanguage')" prop="selectLanguage"
        v-if="langItemList && langItemList.length > 1">
        <el-select v-model="curLang" multiple :placeholder="$t('tip.select')" @change='selectLang' style="width: 500px;">
          <el-option v-for="item in langItemList" :key="item.lang" :label="item.name" :value="item.lang">
          </el-option>
        </el-select>
      </el-form-item>
      <!-- 名称 -->
      <div v-for="(item, index) in menuLangList" :key="index">
        <el-form-item
          :label="dataForm.typeList[dataForm.type] + (langItemList && langItemList.length > 1 ? '(' + item.langName + ')' : '')">
          <el-input maxlength="50" size="small" show-word-limit v-model.trim="item.name"></el-input>
        </el-form-item>
      </div>
      <!-- <el-form-item :label="dataForm.typeList[dataForm.type] + $t('publics.name')" prop="name">
        <el-input
          v-model="dataForm.name"
          maxlength="30"
          size="small"
          show-word-limit
          :placeholder="dataForm.typeList[dataForm.type] + $t('publics.name')"
        ></el-input>
      </el-form-item>
      <el-form-item :label="dataForm.typeList[dataForm.type] + $t('publics.en') + $t('publics.name')">
        <el-input
          v-model="dataForm.nameEn"
          maxlength="50"
          size="small"
          show-word-limit
          :placeholder="dataForm.typeList[dataForm.type] + $t('publics.en') + $t('publics.name')"
        ></el-input>
      </el-form-item> -->
      <el-form-item :label="$t('sys.parentMenu')">
        <el-cascader expand-trigger="hover" :options="menuList" :props="menuListTreeProps" change-on-select
          :clearable="true" size="small" v-model="selectedMenu" @change="handleSelectMenuChange"></el-cascader>
      </el-form-item>
      <el-form-item v-if="dataForm.type === 1" :label="$t('sysManagement.menuRouter')" prop="url">
        <el-input v-model="dataForm.url" :placeholder="$t('sysManagement.menuRouter')" maxlength="100" size="small"
          show-word-limit></el-input>
      </el-form-item>
      <el-form-item :label="$t('sys.isHidden')" prop="hidden">
        <el-select size="small" v-model="dataForm.hidden" @change="() => { isRef = true }"
          :placeholder="$t('user.pleaseSelect')">
          <el-option v-for="item in isHidden" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item v-if="dataForm.type !== 0" :label="$t('sys.authorization')" prop="perms">
        <el-input v-model="dataForm.perms" :placeholder="$t('sys.separated') + 'user:list,user:create'" maxlength="250"
          size="small" show-word-limit></el-input>
      </el-form-item>
      <el-form-item v-if="dataForm.type !== 2" :label="$t('hotSearch.seq')" prop="orderNum">
        <el-input-number v-model="dataForm.orderNum" controls-position="right" :min="0" :label="$t('hotSearch.seq')"
          maxlength="100" size="small" show-word-limit></el-input-number>
      </el-form-item>
      <el-form-item v-if="dataForm.type !== 2" :label="$t('sys.menuIcon')" prop="icon">
        <el-row>
          <el-col :span="22">
            <el-popover ref="iconListPopover" placement="bottom-start" trigger="click"
              popper-class="mod-menu__icon-popover">
              <div class="mod-menu__icon-list" style="height: 280px !important">
                <el-button v-for="(item, index) in iconList" :key="index" @click="iconActiveHandle(item)"
                  :class="{ 'is-active': item === dataForm.icon }">
                  <icon-svg style="stroke: #8A979E !important;color: #8A979E" :name="item"></icon-svg>
                </el-button>
              </div>
            </el-popover>
            <el-input v-model="dataForm.icon" v-popover:iconListPopover maxlength="30" size="small" show-word-limit
              :placeholder="$t('sys.menuIconName')" class="icon-list__input" clearable></el-input>
          </el-col>
          <el-col :span="2" class="icon-list__tips">
            <el-tooltip placement="top" effect="light">
              <div slot="content">{{ $t('sys.content') }}</div>
              <i class="el-icon-warning"></i>
            </el-tooltip>
          </el-col>
        </el-row>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <div class="default-btn" @click="close()">{{ $t('coupon.cancel') }}</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">{{ $t('coupon.confirm') }}</div>
    </span>
  </el-dialog>
</template>

<script>
import { treeDataTranslate, idList } from '@/utils'
import Icon from '@/icons'
export default {
  data() {
    var validateUrl = (rule, value, callback) => {
      if (this.dataForm.type === 1 && !/\S/.test(value)) {
        callback(new Error(this.$i18n.t('sys.menuUrlNoNull')))
      } else {
        callback()
      }
    }
    // var validateName = (rule, value, callback) => {
    //   if (!value.trim()) {
    //     callback(new Error(this.$i18n.t('sys.menuNameNoNull')))
    //   } else {
    //     callback()
    //   }
    // }
    return {
      visible: false,
      isRef: false,
      dataForm: {
        id: 0,
        type: 1,
        typeList: [this.$i18n.t('sys.catalog'), this.$i18n.t('sys.menu'), this.$i18n.t('sys.button')],
        parentId: 0,
        url: '',
        perms: '',
        orderNum: 0,
        icon: '',
        iconList: [],
        hidden: 0,
        menuLangList: []
      },
      dataRule: {
        // name: [
        //   { required: true, message: this.$i18n.t('sys.menuNameNoNull'), trigger: 'blur' },
        //   { validator: validateName, required: true, trigger: 'blur' }
        // ],
        url: [
          { validator: validateUrl, required: true, trigger: 'blur' }
        ],
        hidden: [
          { required: true, message: this.$i18n.t('sys.menuNameNoNull'), trigger: 'blur' }
        ]
      },
      isSubmit: false,
      menuList: [],
      selectedMenu: [],
      menuListTreeProps: {
        value: 'menuId',
        label: 'name'
      },
      isHidden: [
        {
          label: this.$i18n.t('sys.no'),
          value: 0
        },
        {

          label: this.$i18n.t('sys.yes'),
          value: 1
        }
      ],
      // 语言列表
      langItemList: [],
      masterLangInfo: { name: '', lang: '' },
      // 当前所选语言
      curLang: [],
      menuLangList: []
    }
  },
  watch: {
    'dataForm.type': {
      deep: true,
      handler(newV, oldV) {
        this.$refs['dataForm'].clearValidate()
      }
    }
  },
  created() {
    this.iconList = Icon.getNameList()
  },
  methods: {
    init(id) {
      this.getLangList()
      this.dataForm.id = id || 0
      this.isSubmit = false
      this.dataForm.name = null
      this.$http({
        url: this.$http.adornUrl('/sys/menu/list'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        this.menuList = treeDataTranslate(data, 'menuId')
      }).then(() => {
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
        })
      }).then(() => {
        if (this.dataForm.id) {
          // 修改
          this.$http({
            url: this.$http.adornUrl(`/sys/menu/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
            this.dataForm.id = data.menuId
            this.dataForm.type = data.type
            this.dataForm.parentId = data.parentId
            this.dataForm.url = data.url
            this.dataForm.perms = data.perms
            this.dataForm.orderNum = data.orderNum
            this.dataForm.icon = data.icon
            this.dataForm.hidden = data.hidden ? 1 : 0
            this.dataForm.menuLangList = data.menuLangList
            this.menuLangList = data.menuLangList
            this.selectedMenu = idList(this.menuList, data.parentId, 'menuId', 'children').reverse()
            this.getLangList()
          })
        } else {
          this.selectedMenu = []
          this.getLangList()
        }
      })
    },
    /**
     * 获取商品配置的语言列表
     */
    getLangList() {
      this.$http({
        url: this.$http.adornUrl('/sys/lang'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        console.log('初始化国际化配置', data)
        if (data) {
          const info = data
          console.log('langConfig', info)
          this.masterLangInfo.name = info.name
          this.masterLangInfo.lang = info.lang
          this.langItemList = info.langItemList

          if (this.dataForm.id) {
            let curLang = []
            let menuLangList = []
            for (const item of this.langItemList) {
              const fd = this.menuLangList.find(it => it.lang === item.lang)
              if (fd) {
                fd.langName = item.name
                menuLangList.push(fd)
                curLang.push(item.lang)
              }
            }
            this.menuLangList = menuLangList
            this.selectLang(curLang)
          } else {
            // 设置默认主语言
            this.selectLang([info.lang])
          }
        }
      })
    },
    selectLang(value) {
      console.log('当前值', value)
      this.curLang = value
      // 设置主语言不可删除
      if (!this.curLang.includes(this.masterLangInfo.lang)) {
        this.curLang.unshift(this.masterLangInfo.lang)
      }
      // 菜单名称
      const tempArr = this.menuLangList.filter(item => this.curLang.includes(item.lang))
      this.curLang.forEach((item, index) => {
        if (!tempArr.find(f => f.lang == item)) {
          const fd = this.langItemList.find(it => it.lang === item)
          if (fd) {
            tempArr.splice(index, 0, { langName: fd.name, lang: item, name: '' })
          }
        }
      })
      this.menuLangList = tempArr
    },
    handleSelectMenuChange(val) {
      if (val.length > 0) {
        this.dataForm.parentId = val[val.length - 1]
      }else{
        this.dataForm.parentId=0
      }
    },
    clear() {
      this.dataForm = {
        id: 0,
        type: 1,
        typeList: [this.$i18n.t('sys.catalog'), this.$i18n.t('sys.menu'), this.$i18n.t('sys.button')],
        name: '',
        nameEn: '',
        parentId: 0,
        url: '',
        perms: '',
        orderNum: 0,
        icon: '',
        iconList: [],
        hidden: 0
      }
    },
    // 图标选中
    iconActiveHandle(iconName) {
      this.dataForm.icon = iconName
    },
    beforeClose(done) {
      this.clear()
      done()
    },
    close() {
      this.clear()
      this.visible = false
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.isSubmit) {
            return false
          }
          for (var i = 0; i < this.menuLangList.length; i++) {
            if (!this.menuLangList[i].name) {
              this.errorMsg(this.dataForm.typeList[this.dataForm.type] + this.$i18n.t('sys.menuNameNoNull'))
              return false
            }
          }

          this.isSubmit = true
          console.log(this.dataForm.parentId)
          this.$http({
            url: this.$http.adornUrl(`/sys/menu`),
            method: this.dataForm.id ? 'put' : 'post',
            data: this.$http.adornData({
              'menuId': this.dataForm.id || undefined,
              'type': this.dataForm.type,
              'parentId': this.dataForm.parentId,
              'url': this.dataForm.type === 1 ? this.dataForm.url : '',
              'perms': this.dataForm.type === 0 ? '' : this.dataForm.perms,
              'orderNum': this.dataForm.orderNum,
              'icon': this.dataForm.type === 2 ? '' : this.dataForm.icon,
              'hidden': this.dataForm.hidden,
              'menuLangList': this.menuLangList
            })
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
                this.clear()
                if (this.isRef) {
                  location.reload()
                }
              }
            })
          }).catch((e) => {
            this.isSubmit = false
          })
        }
      })
    },
    errorMsg(message) {
      this.$message({
        message: message,
        type: 'error',
        duration: 1500,
        customClass: 'zZindex'
      })
    }
  }
}
</script>

<style lang="scss">
.mod-menu {

  .menu-list__input,
  .icon-list__input {
    >.el-input__inner {
      cursor: pointer;
    }
  }

  &__icon-popover {
    max-width: 420px;
  }

  &__icon-list {
    max-height: 280px;
    padding: 0;
    margin: -8px 0 0 -8px;

    >.el-button {
      padding: 8px;
      margin: 8px 0 0 8px;

      >span {
        display: inline-block;
        vertical-align: middle;
        width: 18px;
        height: 18px;
        font-size: 18px;
      }
    }
  }

  .icon-list__tips {
    font-size: 18px;
    text-align: center;
    color: #e6a23c;
    cursor: pointer;
  }
}
</style>
