<template>
    <div>

        <el-container>
            <el-aside width="auto">
                <el-menu class="el-menu-vertical" :collapse="isCollapse" @open="collapseOpen" @close="collapseClose">
                    <el-menu-item index="1">
                        <el-menu-item index="1">
                            <i class="el-icon-user"></i>
                            <span slot="title">牛二黑</span>
                        </el-menu-item>
                        <el-menu-item index="2">
                            <i class="el-icon-user"></i>
                            <span slot="title">大壮</span>
                        </el-menu-item>
                        <el-menu-item index="3">
                            <i class="el-icon-user"></i>
                            <span slot="title">导航二</span>
                        </el-menu-item>
                    </el-menu-item>
                </el-menu>
            </el-aside>
            <el-container>
                <el-header>
                    <button @click="collapseStatus">
                        <i class="el-icon-s-unfold"></i>
                    </button>
                </el-header>
                <el-main>Main</el-main>
                <el-footer>Footer</el-footer>
            </el-container>
        </el-container>


    </div>
</template>

<script>
export default {
    name: "MessageUI4",
    data() {
        return {
            isCollapse: true,
        }
    },
    methods: {
        collapseStatus() {
            this.isCollapse = !this.isCollapse;
        },
        collapseOpen() {
            this.isCollapse = false;
        },
        collapseClose() {
            this.isCollapse = true;
        }
    },



}
</script>

<style scoped>
.el-menu-vertical:not(.el-menu--collapse) {
    width: 200px;
    min-height: 400px;
}

.el-main {
    background-color: #E9EEF3;
    color: #333;
    text-align: center;
    line-height: 160px;
}
.el-aside {
    background-color: #D3DCE6;
    color: #333;
    text-align: center;
    line-height: 200px;
}
</style>