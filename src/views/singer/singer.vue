<template>
    <div>
        <input type="button"
               value="+带数据数据"
               @click="add(5)">
        <hr />
        <input type="button"
               value="+默认加1提交"
               @click="increment">
        <br />
        <span>当前的值：{{count}}</span>
        <hr />
        <el-button type="success"
                   @click="downloadFn1">excel文件的下载1</el-button>
        <el-button type="success"
                   @click="uploadFn2">excel文件的上传2</el-button>

        <input id="upload"
               style="visibility: hidden;"
               type="file"
               ref="uploadExcel"
               @change="importfn($event)"
               accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel" />

        <hr>

        <MyCard :list="listData">
            <div slot="title">这是父组件自定义分发的内容</div>
        </MyCard>

        <hr>

        <template>
            <el-table :data="tableData"
                      v-loading="isShowLoading"
                      border
                      style="width: 100%">
                <el-table-column fixed
                                 prop="date"
                                 label="日期"
                                 width="150">
                </el-table-column>
                <el-table-column prop="name"
                                 label="姓名"
                                 width="120">
                </el-table-column>
                <el-table-column prop="province"
                                 label="省份"
                                 width="120">
                </el-table-column>
                <el-table-column prop="city"
                                 label="市区"
                                 width="120">
                </el-table-column>

                <el-table-column label="操作"
                                 width="100">
                    <template slot-scope="scope">
                        <el-button @click="upFieldOrder(scope.$index)"
                                   type="text"
                                   size="small">上移</el-button>
                        <el-button type="text"
                                   @click="downFieldOrder(scope.$index)"
                                   size="small">下移</el-button>
                    </template>
                </el-table-column>

            </el-table>
        </template>

    </div>
</template>
<script>
import { mapActions, mapState } from 'vuex'
import { getCookie } from "@/utils/index.js"
import fileServer from "file-saver";
import XLSX from "xlsx";
import MyCard from "@/components/card.vue"
import MinXin from '@/components/MinXins/index.js'

export default {
    mixins: [MinXin],
    data () {
        return {
            params: {
                template_id: 0,
                type: 0,
                page: 1,
                pagesize: 10,
                down: 0,// 0默认  1下载
                startdate: "2018-10-21",
                enddate: '2018-10-21',

            },
            listData: [
                "蓝色",
                "橘色",
                "橙色"
            ],
            tableData: null
        }
    },
    components: {
        "MyCard": MyCard
    },
    beforeRouteLeave (to, from, next) {
        // 设置下一个路由的 meta
        to.meta.keepAlive = false; // C 跳转到 A 时让 A 不缓存，即刷新
        next();
    },
    methods: {//mapActions
        getList () {
            setTimeout(() => {//模拟ajax的请求数据
                this.tableData = [{
                    date: '2016-05-01',
                    name: '王小虎1',
                    province: '上海',
                    city: '普陀区',
                    address: '上海市普陀区金沙江路 1518 弄',
                    zip: 200333
                }, {
                    date: '2016-05-02',
                    name: '王小虎2',
                    province: '上海',
                    city: '普陀区',
                    address: '上海市普陀区金沙江路 1518 弄',
                    zip: 200333
                }, {
                    date: '2016-05-03',
                    name: '王小虎3',
                    province: '上海',
                    city: '普陀区',
                    address: '上海市普陀区金沙江路 1518 弄',
                    zip: 200333
                }]
                this.isShowLoading = false

            }, 3000)
        },

        upFieldOrder (index) {
            let temp = this.tableData[index - 1];
            this.$set(this.tableData, index - 1, this.tableData[index])
            this.$set(this.tableData, index, temp)
        },
        downFieldOrder (index) {//下移元素
            let temp = this.tableData[index];//Vue的set实现实时的更新的状态
            this.$set(this.tableData, index, this.tableData[index + 1])
            this.$set(this.tableData, index + 1, temp)
        },
        ...mapActions(['increment']),
        //在此处的时候。increasement相当于是一个函数
        //action提交额外数据的时候需要手动调用  mapAction提交数据的时候，需要可以进行传递参数的形式
        //mapaction相当于是一个函数 可以相互的调用

        add (num) {
            this.increment({
                n: num
            })
        },

        //    this.$store.dispatch("increment",{
        //      n:num
        //    })

        importfn (event) {
            // console.log(event.currentTarget.files[0])
            let _this = this;
            let inputDOM = this.$refs.inputer;
            // 通过DOM取文件数据
            this.file = event.currentTarget.files[0];
            var rABS = false; //是否将文件读取为二进制字符串
            var f = this.file;
            var reader = new FileReader();
            //if (!FileReader.prototype.readAsBinaryString) {
            FileReader.prototype.readAsBinaryString = function (f) {
                var binary = "";
                var rABS = false; //是否将文件读取为二进制字符串
                var pt = this;
                var wb; //读取完成的数据
                var outdata;
                var reader = new FileReader();
                reader.onload = function (e) {
                    var bytes = new Uint8Array(reader.result);
                    var length = bytes.byteLength;
                    for (var i = 0; i < length; i++) {
                        binary += String.fromCharCode(bytes[i]);
                    }
                    var XLSX = require('xlsx');
                    if (rABS) {
                        wb = XLSX.read(btoa(fixdata(binary)), { //手动转化
                            type: 'base64'
                        });
                    } else {
                        wb = XLSX.read(binary, {
                            type: 'binary'
                        });
                    }
                    outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]);
                    console.log(outdata)
                    //outdata就是你想要的东西<br>　　
                    //let importList = _this.dateTransition(outdata);
                }
                reader.readAsArrayBuffer(f);
            }
            if (rABS) {
                reader.readAsArrayBuffer(f);
            } else {
                reader.readAsBinaryString(f);
            }
        },

        //将对应的中文key转化为自己想要的英文key
        dateTransition (outdata) {
            let list = [];
            var obj = {};
            for (var i = 0; i < outdata.length; i++) {
                obj = {};
                for (var key in outdata[i]) {
                    if (key == '工号') {
                        obj['jobNumber'] = outdata[i][key];
                    } else if (key == '姓名') {
                        obj['name'] = outdata[i][key];
                    } else if (key == '部门') {
                        obj['department'] = outdata[i][key];
                    }
                }
                list.push(obj);
            }
            return list;
        },

        downloadFn1 () {
            this.params.down = 1;
            this.$http({
                method: "get",
                //  url: "/api/datatemp-show",
                url: `http://localhost:8089/static/down1.json`,
                data: this.params
            }).then(
                response => {
                    var res = response.data;
                    if (res.code == 0) {
                        this.downloadCSV(res.data, this.params.startdate + "至" + this.params.enddate);
                    }
                },
                function (error) { }
            );
        },

        downloadCSV (val, filename) {
            function datenum (v, date1904) {
                if (date1904) v += 1462;
                var epoch = Date.parse(v);
                return (
                    (epoch - new Date(Date.UTC(1899, 11, 30))) / (24 * 60 * 60 * 1000)
                );
            }
            function sheet_from_array_of_arrays (data, opts) {
                var ws = {};
                var range = {
                    s: {
                        c: 10000000,
                        r: 10000000
                    },
                    e: {
                        c: 0,
                        r: 0
                    }
                };
                for (var R = 0; R != data.length; ++R) {
                    for (var C = 0; C != data[R].length; ++C) {
                        if (range.s.r > R) range.s.r = R;
                        if (range.s.c > C) range.s.c = C;
                        if (range.e.r < R) range.e.r = R;
                        if (range.e.c < C) range.e.c = C;
                        var cell = {
                            v: data[R][C]
                        };
                        if (cell.v == null) continue;
                        var cell_ref = XLSX.utils.encode_cell({
                            c: C,
                            r: R
                        });
                        if (typeof cell.v === "number") cell.t = "n";
                        else if (typeof cell.v === "boolean") cell.t = "b";
                        else if (cell.v instanceof Date) {
                            cell.t = "n";
                            cell.z = XLSX.SSF._table[14];
                            cell.v = datenum(cell.v);
                        } else cell.t = "s";
                        ws[cell_ref] = cell;
                    }
                }
                if (range.s.c < 10000000) ws["!ref"] = XLSX.utils.encode_range(range);
                return ws;
            }
            function Workbook () {
                if (!(this instanceof Workbook)) return new Workbook();
                this.SheetNames = [];
                this.Sheets = {};
            }
            var key = XLSX.utils.encode_cell({
                c: 0,
                r: 0
            });
            var ws = sheet_from_array_of_arrays(val);
            var workbook = new Workbook();
            workbook.SheetNames.push("kdg");
            workbook.Sheets["kdg"] = ws;
            var wbout = XLSX.write(workbook, {
                bookType: "xlsx",
                bookSST: true,
                type: "binary"
            });
            function s2ab (s) {
                var buf = new ArrayBuffer(s.length);
                var view = new Uint8Array(buf);
                for (var i = 0; i != s.length; ++i) view[i] = s.charCodeAt(i) & 0xff;
                return buf;
            }
            fileServer.saveAs(
                new Blob([s2ab(wbout)], {
                    type: ""
                }),
                filename + ".xlsx"
            );
        },

        //文件上传方式
        uploadFn2 () {
            //console.log(this.$refs.uploadExcel) 
            this.$refs.uploadExcel.click();//一个元素触发另一个元素的点击
        },

    },
    computed: {
        ...mapState([
            "count"
        ]),
    },
    mounted () {
        //适当的将变量进行存储，就行优化的方便查找
        let username = getCookie("username") || "";
        if (username) {
            //  alert(username)
        }
        this.getList()
    }

}
</script>

