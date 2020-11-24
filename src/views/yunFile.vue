<template>
    <div ref="wrap" class="main">
        <!-- 云盘 -->
        <div class="yun-file">
            <div class="pb20">
                <div class="yun-file-title flex" v-bind:style="{'height': $isIPad ? '47wx': '94px'}">
                    <div class="title flex">
                        <text class="c06 f30">{{i18n.RecentlyUsedFile}}</text>
                    </div>
                    <bui-image src="/image/more.png" width="18wx" height="18wx" @click="yunFileMoreEvent"></bui-image>
                </div>
                <div class="prl36">
                    <div v-if='isShowRE'>
                        <div v-if='yunFileReleArr.length != 0'>
                            <div class="flex-dr flex-ac flex-sb" v-for="(item, index) in yunFileReleArr" :key='index' @click='yunFileUserEvent(item.id, item.name, item.isExitDoc, item.dir)' v-bind:style="{'height': $isIPad ? '64wx': '128px'}">
                                <div class="flex-dr flex-ac">
                                    <bui-image :src="item.image" width="26wx" height="26wx" radius='10px' @click='yunFileUserEvent(item.id, item.name, item.isExitDoc, item.dir)'></bui-image>
                                    <div class="pl28">
                                        <text class="f32 c0 fw4 lines1">{{item.name}}</text>
                                        <text class="f24 c45 fw4 lines1 mt16" v-if="item.size">{{item.size}}</text>
                                    </div>
                                </div>
                                <div>
                                    <text class="f24 c45 fw4 mt16" v-if="item.time">{{item.time}}</text>
                                </div>
                            </div>
                        </div>
                        <div class="no-file flex-ac flex-jc" v-if='yunFileReleArr.length == 0'>
                            <bui-image src="/image/nodata.png" width="109wx" height="70wx" style="margin-bottom: 16wx"></bui-image>
                            <text class="f32 c45 fw4 pl15">{{isErrorRele?i18n.NoneData:i18n.ErrorLoadData}}</text>
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
const globalEvent = weex.requireModule('globalEvent');
const navigator = weex.requireModule('navigator');
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
            urlParams: {}
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
        if (this.$isAndroid()) {
            globalEvent.addEventListener("androidback", function (e) {
                navigator.close()
            });
        }
    },
    methods: {
        getStorage(callback) {
            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
            storage.getItem('shareMebig20201124' + ecode + pageId, res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    this.yunFileReleArr = data
                    this.isShowRE = true
                    this.isErrorRele = true
                    this.broadcastWidgetHeight()
                    setTimeout(() => {
                        this.getYunFile()
                    }, 2000)
                } else {
                    callback()
                }
            })
        },
        yunFileMoreEvent() {
            link.launchLinkService(['[OpenBuiltIn] \n key=ShareToMeList'], (res) => { }, (err) => { });
        },
        yunFileUserEvent(id, name, isExitDoc, item) {
            if (!isExitDoc) {
                link.startSharedDirectory([{ model: item }])
                return
            }
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
        getYunFile() {
            link.getServerConfigs([], (params) => {
                link.getLoginInfo([], (user) => {
                    let url = params.diskUrl ? params.diskUrl : params.diskUri
                    let objDataTo = {
                        by: '',
                        to: 'U' + user.userId,
                        scope: '',
                        limit: 3
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
                                fileObj['size'] = element.size ? this.conver(element.size) : ''
                                fileObj['time'] = this.format(element.createdTime, 'MM-dd')
                                if (element.type == 'D') {
                                    fileObj['isExitDoc'] = false
                                    fileObj['dir'] = element
                                    fileObj['image'] = '/image/folder2.png'
                                } else {
                                    fileObj['isExitDoc'] = true
                                    fileObj['image'] = this.getFileImages(element.extension)
                                }
                                fileArr.push(fileObj)
                            }
                            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
                            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
                            this.yunFileReleArr = fileArr
                            storage.setItem('shareMebig20201124' + ecode + pageId, JSON.stringify(fileArr))
                        } catch (error) {
                            this.isShowRE = true
                            this.isErrorRele = false
                            this.yunFileReleArr = []
                        }
                        this.broadcastWidgetHeight()
                    }, (err) => {
                        this.error()
                    })
                }, (err) => {
                    this.error()
                })
            }, () => {
                this.error()
            });
        },
        format(ts, fmt) {
            if (!ts) return '';
            if (!fmt) fmt = 'yyyy/MM/dd hh:mm:ss'
            var dt = new Date(ts)
            var o = {
                'M+': dt.getMonth() + 1, // 月份
                'd+': dt.getDate(), // 日
                'h+': dt.getHours(), // 小时
                'm+': dt.getMinutes(), // 分
                's+': dt.getSeconds(), // 秒
                'q+': Math.floor((dt.getMonth() + 3) / 3), // 季度
                'S': dt.getMilliseconds() // 毫秒
            }
            if (/(y+)/.test(fmt)) {
                let $1 = fmt.split('-')[0]
                fmt = fmt.replace(RegExp.$1, (dt.getFullYear() + '').substr(4 - RegExp.$1.length))
            }
            for (var k in o) {
                if (new RegExp('(' + k + ')').test(fmt)) {
                    fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? (o[k]) : (('00' + o[k]).substr(('' + o[k]).length)))
                }
            }
            return fmt
        },
        conver(limit) {
            var size = "";
            if (limit < 0.1 * 1024) { //如果小于0.1KB转化成B  
                size = limit.toFixed(2) + "B";
            } else if (limit < 0.1 * 1024 * 1024) {//如果小于0.1MB转化成KB  
                size = (limit / 1024).toFixed(2) + "KB";
            } else if (limit < 0.1 * 1024 * 1024 * 1024) { //如果小于0.1GB转化成MB  
                size = (limit / (1024 * 1024)).toFixed(2) + "MB";
            } else { //其他转化成GB  
                size = (limit / (1024 * 1024 * 1024)).toFixed(2) + "GB";
            }

            var sizestr = size + "";
            var len = sizestr.indexOf("\.");
            var dec = sizestr.substr(len + 1, 2);
            if (dec == "00") {//当小数点后为00时 去掉小数部分  
                return sizestr.substring(0, len) + sizestr.substr(len + 3, 2);
            }
            return sizestr;
        },
        error() {
            this.yunFileReleArr = []
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
.main {
}

.yun-file {
    background-color: #fff;
}

.yun-file-title {
    padding: 0 18wx;
}

.no-file {
    height: 192wx;
}
</style>