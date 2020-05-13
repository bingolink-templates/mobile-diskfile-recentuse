<template>
    <div ref="wrap">
        <!-- 云盘 -->
        <div class="yun-file">
            <div class="pb20">
                <div class="yun-file-title flex" v-bind:style="{'height': $isIPad ? '44wx': '88px', 'margin-bottom': $isIPad ? '7wx': '15px'}">
                    <div class="title flex">
                        <span class="line" v-bind:style="{'background-color': themeColor}"></span>
                        <text class="c0 f30">{{i18n.RecentlyUsedFile}}</text>
                    </div>
                    <text class="f24 c153 fw4 pl20 pt10 pb10" @click="yunFileMoreEvent">{{i18n.All}}</text>
                </div>
                <div class="prl30">
                    <div v-if='isShowRE'>
                        <div v-if='yunFileReleArr.length != 0' class='disk' v-bind:style="{'height': $isIPad ? '200wx': '400px'}">
                            <div class="flex-dr flex-ac" v-if='item.isExitDoc' v-for="(item, index) in yunFileReleArr" :key='index' @click='yunFileUserEvent(item.id)' v-bind:style="{'height': $isIPad ? '40wx': '80px'}">
                                <bui-image :src="item.image" width="26wx" height="26wx" radius='10px'  @click='yunFileUserEvent(item.id)'></bui-image>
                                <text class="f24 c51 fw4 pl20 lines1">{{item.name}}</text>
                            </div>
                        </div>
                        <div class="no-file flex-ac flex-jc" v-if='yunFileReleArr.length == 0'>
                            <div class="flex-dr">
                                <bui-image src="/image/sleep1.png" width="21wx" height="21wx"></bui-image>
                                <text class="f26 c51 fw4 pl15 center-height ">{{isErrorRele?i18n.NoneData:i18n.ErrorLoadData}}</text>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
const link = weex.requireModule("LinkModule");
const linkapi = require("linkapi");
const dom = weex.requireModule('dom');
const util = require('ser/util')
const storage = weex.requireModule('storage');
export default {
    data() {
        return {
            yunFileReleArr: [],
            isErrorRele: true,
            isShowRE: false,
            channel: new BroadcastChannel('WidgetsMessage'),
            i18n: '',
            themeColor: '',
            $isIPad: false,
        }
    },
    created() {
        this.$fixViewport();
        linkapi.getLanguage((res) => {
            this.i18n = this.$window[res]
        })
        linkapi.getThemeColor(res => {
            this.themeColor = res.background_color;
        })
        this.$isIPad = this.$isIPad()
    },
    mounted() {
        this.channel.onmessage = (event) => {
            if (event.data.action === 'RefreshData') {
                this.getYunFile()
            }
        }
        this.getStorage(() => {
            this.getYunFile()
        })
    },
    methods: {
        getStorage(callback) {
            storage.getItem('yunFileJLocalData2020513', res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    this.yunFileReleArr = data
                    this.isShowRE = true
                    this.isErrorRele = true
                    this.broadcastWidgetHeight()
                } else {
                    callback()
                }
            })
        },
        yunFileMoreEvent() {
            link.launchLinkService(['[OpenBuiltIn] \n key=MyDisk'], (res) => { }, (err) => { });
        },
        yunFileUserEvent(id) {
            link.launchLinkService(['[OpenBuiltIn] \n key=DiskDetail \n diskId=' + id], (res) => { }, (err) => { });
        },
        getFileImages(ext) {
            let fileImageTypes = {
                'excel': ['xls', 'xlsx'],
                'music': ['mp3', 'wma', 'wav', 'mod', 'ogg', 'm4a'],
                'pdf': ['pdf'],
                'photo': ['bmp', 'gif', 'jpeg', 'jpg', 'svg', 'png', 'psd'],
                'ppt': ['ppt', 'pptx'],
                'txt': ['txt', 'key'],
                'video': ['rm', 'rmvb', 'wmv', 'avi', 'mp4', '3gp', 'mkv', 'flv', 'mov', 'mpg'],
                'word': ['doc', 'docx', 'wps'],
                'zip': ['zip', 'rar', '7z'],
                'unknow': ['file'],
                'folder2': ['folder'],
                'team': ['team']
            }
            let fileImages = {};
            let fileTypeImages = {};
            for (let fext in fileImageTypes) {
                fileImages[fext] = '/image/' + fext + '.png';
                let arr = fileImageTypes[fext];
                if (arr.length > 0) {
                    for (let i = 0; i < arr.length; i++) {
                        fileTypeImages[arr[i]] = fext;
                    }
                }
            }
            let type = fileTypeImages[ext];
            if (type) {
                return fileImages[type];
            } else {
                return fileImages['unknow'];
            }
        },
        getYunFile() {
            link.getServerConfigs([], (params) => {
                link.getLoginInfo([], (user) => {
                    let url = params.diskUrl ? params.diskUrl : params.diskUri
                    linkapi.get({
                        url: url + '/openapi/file/list?path=%2FPrivate&bounds=%7B%22offset%22%3A0%2C%22limit%22%3A200%7D',
                    }).then((res) => {
                        try {
                            this.isShowRE = true
                            this.isErrorRele = true
                            let fileArr = [], files = [];
                            for (let index = 0; index < res.rows.length; index++) {
                                let fileObj = {}
                                const element = res.rows[index];
                                fileObj['name'] = element.name
                                fileObj['id'] = element.id
                                if (element.type == 'D') {
                                    fileObj['isExitDoc'] = false
                                    fileObj['image'] = '/image/folder2.png'
                                } else {
                                    fileObj['isExitDoc'] = true
                                    fileObj['image'] = this.getFileImages(element.extension)
                                }
                                fileArr.push(fileObj)
                            }
                            this.yunFileReleArr = fileArr
                            storage.setItem('yunFileJLocalData2020513', JSON.stringify(fileArr))
                        } catch (error) {
                        }
                        this.broadcastWidgetHeight()
                    }, (err) => {
                        this.isShowRE = true
                        this.isErrorRele = false
                        this.broadcastWidgetHeight()
                    })
                }, (err) => {
                    this.error()
                })
            }, () => {
                this.error()
            });
        },
        error() {
            this.isShowRE = true
            this.isErrorRele = false
            this.broadcastWidgetHeight()
        },
        getComponentRect(_params) {
            dom.getComponentRect(this.$refs.wrap, (ret) => {
                this.channel.postMessage({
                    widgetHeight: ret.size.height,
                    id: _params.id
                });
            });
        },
        broadcastWidgetHeight() {
            let _params = this.$getPageParams();
            // 防止高度通知失败
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 200)
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 1200)
        }
    }
}
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.yun-file {
    background-color: #fff;
}

.line {
    width: 5px;
    height: 36px;
    margin-right: 12px;
}

.yun-file-title {
    padding: 0 12wx;
    border-bottom: 1px solid #f2f2f2;
}

.no-file {
    height: 70wx;
    flex: 1;
}

.center-height {
    line-height: 20wx;
}
.disk {
    overflow: hidden;
}
</style>