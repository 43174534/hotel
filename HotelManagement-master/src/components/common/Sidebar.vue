<template>
    <div class="sidebar">
        <el-menu
            class="sidebar-el-menu"
            :default-active="onRoutes"
            :collapse="collapse"
            background-color="#324157"
            text-color="#bfcbd9"
            active-text-color="#20a0ff"
            unique-opened
            router
        >
            <template v-for="item in items">
                <template v-if="item.subs">
                    <el-submenu :index="item.index" :key="item.index">
                        <template slot="title">
                            <i :class="item.icon"></i>
                            <span slot="title">{{ item.menuName }}</span>
                        </template>
                        <!-- <template v-for="subItem in item.subs">
                            <el-submenu
                                v-if="subItem.subs"
                                :index="subItem.index"
                                :key="subItem.index"
                            >
                                <template slot="title">{{ subItem.title }}</template>
                                <el-menu-item
                                    v-for="(threeItem,i) in subItem.subs"
                                    :key="i"
                                    :index="threeItem.index"
                                >{{ threeItem.title }}</el-menu-item>
                            </el-submenu>
                            <el-menu-item
                                v-else
                                :index="subItem.index"
                                :key="subItem.index"
                            >{{ subItem.title }}</el-menu-item>
                        </template> -->
                    </el-submenu>
                </template>
                <template v-else>
                    <el-menu-item :index="item.index" :key="item.index">
                        <i :class="item.icon"></i>
                        <span slot="title">{{ item.menuName }}</span>
                    </el-menu-item>
                </template>
            </template>
        </el-menu>
    </div>
</template>

<script>
import bus from '../common/bus';
import { getLoginEmployee } from '@/api/login'
export default {
    data() {
        return {
            collapse: false,
            items: [
                {
                    icon: 'el-icon-lx-home',
                    index: 'dashboard',
                    menuName: '系统首页'
                },
                {
                    icon: 'el-icon-lx-cascades',
                    index: 'employee',
                    menuName: '员工管理'
                },
                 {
                    icon: 'el-icon-lx-cascades',
                    index: 'users',
                    menuName: '用户管理'
                },
                 {
                    icon: 'el-icon-lx-calendar',
                    index: 'room',
                    menuName: '客房管理'
                },
                {
                    icon: 'el-icon-lx-cascades',
                    index: 'engage',
                    menuName: '预订订单管理'
                },
                 {
                    icon: 'el-icon-lx-cascades',
                    index: 'order',
                    menuName: '订单管理'
                },
                 {
                    icon: 'el-icon-lx-cascades',
                    index: 'historyorder',
                    menuName: '历史订单管理'
                },

                
                {
                    icon: 'el-icon-lx-copy',
                    index: 'cropper',
                    menuName: '文件上传(可裁剪)'
                },
            ]
        };
    },
    computed: {//员工管理、客房管理、客房查询、入住管理、客户服务、财务管理
        onRoutes() {
            return this.$route.path.replace('/', '');
        }
    },
    created() {
        // 通过 Event Bus 进行组件间通信，来折叠侧边栏
        bus.$on('collapse', msg => {
            this.collapse = msg;
            bus.$emit('collapse-content', msg);
        });
    },
    mounted() {
        // this.getLoginEmployee()
    },
    methods: {
        // getLoginEmployee() {
        //     getLoginEmployee().then(res => {
        //         if(res.data.code == 0){
        //             // console.log('当前登录的菜单是',res.data.data.menus)
        //             let menus = res.data.data.menus
        //             for (let i = 0; i < menus.length; i++){
        //                 if(menus[i].menuName == '系统首页') {
        //                     menus[i].index = 'dashboard'
        //                     menus[i].icon = 'el-icon-lx-home'
        //                 } else if(menus[i].menuName == '员工管理') {
        //                     menus[i].index = 'employee'
        //                     menus[i].icon = 'el-icon-lx-cascades'
        //                 } 
        //             }
        //             this.items = menus
        //         }
        //     })
        // }
    }
};
</script>

<style scoped>
.sidebar {
    display: block;
    position: absolute;
    left: 0;
    top: 70px;
    bottom: 0;
    overflow-y: scroll;
}
.sidebar::-webkit-scrollbar {
    width: 0;
}
.sidebar-el-menu:not(.el-menu--collapse) {
    width: 250px;
}
.sidebar > ul {
    height: 100%;
}
</style>
