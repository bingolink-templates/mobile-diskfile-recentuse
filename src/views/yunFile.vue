<template>
    <div ref="wrap">
        <bui-header v-if='isMore' :title="i18n.RecentlyUsedFile" :leftItem="leftItem" @leftClick="back">
        </bui-header>
        <!-- 云盘 -->
        <div class="yun-file" v-bind:style="{'height': $isIPad ? '302wx': '604px'}">
            <div class="pb20">
                <div v-if='!isMore' class="yun-file-title flex" v-bind:style="{'height': $isIPad ? '44wx': '88px', 'margin-bottom': $isIPad ? '7wx': '15px'}">
                    <div class="title flex">
                        <span class="line" v-bind:style="{'background-color': themeColor}"></span>
                        <text class="c0 f30">{{i18n.RecentlyUsedFile}}</text>
                    </div>
                    <text class="f24 c153 fw4 pl20 pt10 pb10" @click="yunFileMoreEvent">{{i18n.All}}</text>
                </div>
                <div class="prl30" v-bind:style="{'paddingTop': isMore ? '10px': '0'}">
                    <div v-if='isShowRE'>
                        <div v-if='yunFileReleArr.length != 0' v-bind:style="{'height': $isIPad ? '240wx': '480px'}">
                            <div class="flex-dr flex-ac" v-if='item.isExitDoc' v-for="(item, index) in yunFileReleArr" :key='index' @click='yunFileUserEvent(item.id, item.name)' v-bind:style="{'height': $isIPad ? '40wx': '80px'}">
                                <bui-image :src="item.image" width="26wx" height="26wx" radius='10px' @click='yunFileUserEvent(item.id, item.name)'></bui-image>
                                <text class="f24 c51 fw4 pl20 lines1">{{item.name}}</text>
                            </div>
                        </div>
                        <div class="no-file flex-ac flex-jc" v-if='yunFileReleArr.length == 0'>
                            <div class="flex-dr">
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
            urlParams: {},
            noData: false,
            isMore: false,
            leftItem: {
                icon: 'ion-chevron-left',
            },
            empty: [
                {
                    name: '聆客安装指南：注册与安装',
                    id: 'https://www.bingolink.biz/web/faq/themes/themes/index6/index_6_0.html',
                    image: '/image/word.png',
                    isExitDoc: true
                },
                {
                    name: '聆客快速使用指南：销售管理（CRM）',
                    id: 'https://www.bingolink.biz/web/faq/themes/themes/index6/index_6_1.html',
                    image: '/image/word.png',
                    isExitDoc: true
                },
                {
                    name: '聆客快速使用指南：进销存',
                    id: 'https://www.bingolink.biz/web/faq/themes/themes/index6/index_6_2.html',
                    image: '/image/word.png',
                    isExitDoc: true
                },
                {
                    name: '聆客快速使用指南：项目协作',
                    id: 'https://www.bingolink.biz/web/faq/themes/themes/index6/index_6_3.html',
                    image: '/image/word.png',
                    isExitDoc: true
                },
                {
                    name: '聆客快速使用指南：协同办公',
                    id: 'https://www.bingolink.biz/web/faq/themes/themes/index6/index_6_4.html',
                    image: '/image/word.png',
                    isExitDoc: true
                },
                {
                    name: '聆客快速使用指南：高级账款',
                    id: 'https://www.bingolink.biz/web/faq/themes/themes/index6/index_6_5.html',
                    image: '/image/word.png',
                    isExitDoc: true
                }
            ]
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
        this.urlParams = this.resolveUrlParams(weex.config.bundleUrl)
        this.isMore = this.urlParams.isMore || false
    },
    mounted() {
        var that = this
        this.channel.onmessage = (event) => {
            if (event.data.action === 'RefreshData') {
                this.getYunFile()
            }
        }
        this.getStorage(() => {
            that.getYunFile()
        })
    },
    methods: {
        getStorage(callback) {
            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
            storage.getItem('yunFileJLocalDataRecen20205292' + ecode + pageId, res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    if (data.length == 0 && ecode == 'localhost') {
                        this.noData = true
                        this.yunFileReleArr = this.empty
                    } else {
                        this.noData = false
                        this.yunFileReleArr = data
                    }
                    this.isShowRE = true
                    this.isErrorRele = true
                    this.broadcastWidgetHeight()
                } else {
                    callback()
                }
            })
        },
        yunFileMoreEvent() {
            var that = this
            if (this.noData) {
                let runApp = {
                    appCode: 'widget-diskfile-recentuse',
                    data: {
                        isMore: true
                    }
                }
                linkapi.runApp(runApp)
                return
            }
            link.launchLinkService(['[OpenBuiltIn] \n key=ShareToMeList'], (res) => { }, (err) => { });
        },
        yunFileUserEvent(id, name) {
            if (!id) return
            if (id.indexOf('https://') > -1 || id.indexOf('http://') > -1) {
                linkapi.openLinkBroswer(name, id)
            } else {
                link.launchLinkService(['[OpenBuiltIn] \n key=DiskDetail \n diskId=' + id], (res) => { }, (err) => { });
            }
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
        back: function () {
            // 返回上一个页面
            this.$pop();
        },
        getYunFile() {
            link.getServerConfigs([], (params) => {
                link.getLoginInfo([], (user) => {
                    let url = params.diskUrl ? params.diskUrl : params.diskUri
                    let objDataTo = {
                        by: '',
                        to: 'U' + user.userId,
                        scope: '',
                        limit: 6
                    }
                    linkapi.get({
                        url: url + '/openapi/file/share/list',
                        data: objDataTo
                    }).then((res) => {
                        try {
                            this.isShowRE = true
                            this.isErrorRele = true
                            let fileArr = [], files = [];
                            for (let index = 0; index < res.rows.length; index++) {
                                let fileObj = {}
                                const element = res.rows[index];
                                fileObj['name'] = element.name
                                fileObj['id'] = element.fileId || element.id || ''
                                if (element.type == 'D') {
                                    fileObj['isExitDoc'] = false
                                    fileObj['image'] = '/image/folder2.png'
                                } else {
                                    fileObj['isExitDoc'] = true
                                    fileObj['image'] = this.getFileImages(element.extension)
                                }
                                fileArr.push(fileObj)
                            }
                            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
                            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
                            if (fileArr.length == 0 && ecode == 'localhost') {
                                this.noData = true
                                this.yunFileReleArr = this.empty
                            } else {
                                this.noData = false
                                this.yunFileReleArr = fileArr
                            }
                            storage.setItem('yunFileJLocalDataRecen20205292' + ecode + pageId, JSON.stringify(fileArr))
                        } catch (error) {
                            this.isShowRE = true
                            this.isErrorRele = false
                            this.yunFileReleArr = []
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
        resolveUrlParams(url) {
            if (!url) return {};
            url = url + "";
            var index = url.indexOf("?");
            if (index > -1) {
                url = url.substring(index + 1, url.length);
            }
            var pairs = url.split("&"),
                params = {};
            for (var i = 0; i < pairs.length; i++) {
                var pair = pairs[i];
                var indexEq = pair.indexOf("="),
                    key = pair,
                    value = null;
                if (indexEq > 0) {
                    key = pair.substring(0, indexEq);
                    value = pair.substring(indexEq + 1, pair.length);
                }
                params[key] = value;
            }
            return params;
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
    height: 240wx;
    flex: 1;
}

.center-height {
    line-height: 20wx;
}
</style>