<template>
  <el-card style="margin-bottom:20px;">
    <div slot="header" class="clearfix">
      <span>{{ $t('common.aboutMe') }}</span>
    </div>
    <div class="user-profile">
      <div class="box-center">
        <pan-thumb :image="avatar" :height="'100px'" :width="'100px'" :hoverable="false">
          <el-link type="primary" class="change-avatar" @click="dialogVisible = true">{{ $t('common.changeAvatar') }}</el-link>
        </pan-thumb>
      </div>
      <div class="box-center">
        <div class="user-name text-center">{{ user.username }}</div>
        <div class="user-role text-center text-muted">
          <span>{{ user.deptName ? user.deptName : $t('common.noDept') }}</span> · <span>{{ user.roleName ? user.roleName : $t('common.noRole') }}</span>
        </div>
      </div>
    </div>
    <div class="user-bio">
      <div class="user-education user-bio-section">
        <div class="user-bio-section-header"><el-icon class="el-icon-tickets" /><span>{{ $t('table.user.desc') }}</span></div>
        <div class="user-bio-section-body">
          <div class="text-muted">
            {{ user.description ? user.description: $t('tips.nothing') }}
          </div>
        </div>
      </div>
    </div>
    <avatar
      :dialog-visible="dialogVisible"
      @close="dialogVisible = false"
      @success="changeSuccess"
    />
  </el-card>
</template>

<script>
import PanThumb from '@/components/PanThumb'
import Avatar from './Avatar'

export default {
  components: { PanThumb, Avatar },
  props: {
    user: {
      type: Object,
      default: () => {
        return {
        }
      }
    }
  },
  data() {
    return {
      dialogVisible: false
    }
  },
  computed: {
    avatar() {
      return require(`@/assets/avatar/${this.user.avatar}`)
    }
  },
  methods: {
    changeSuccess(avatar) {
      this.dialogVisible = false
      this.$message({
        message: this.$t('tips.updateSuccess'),
        type: 'success'
      })
      this.user.avatar = avatar
      this.$store.commit('account/setUser', this.user)
    }
  }
}
</script>

<style lang="scss" scoped>
 .box-center {
   margin: 0 auto;
   display: table;
 }

 .text-muted {
   color: #777;
 }

 .user-profile {
   .user-name {
     font-weight: bold;
   }

   .box-center {
     padding-top: 10px;
   }

   .user-role {
     padding-top: 10px;
     font-weight: 400;
     font-size: 14px;
   }

   .box-social {
     padding-top: 30px;

     .el-table {
       border-top: 1px solid #dfe6ec;
     }
   }

   .user-follow {
     padding-top: 20px;
   }
 }

 .user-bio {
   margin-top: 20px;
   color: #606266;

   span {
     padding-left: 4px;
   }

   .user-bio-section {
     font-size: 14px;
     padding: 15px 0;

     .user-bio-section-header {
       border-bottom: 1px solid #dfe6ec;
       padding-bottom: 10px;
       margin-bottom: 10px;
       font-weight: bold;
     }
   }
 }
</style>
