<template>

    <div>
        <el-row :gutter="2">
            <el-col :span="4">
                <el-tree :data="treeData"
                         node-key="id"
                         show-checkbox
                         default-expand-all
                         ref="tree"
                         :props="defaultProps">

                </el-tree>
            </el-col>
            <el-col :span=16>
                <el-row :gutter="6">
                    <div v-for="(input, index) in inputs" :key="index" align="bottom">
                        <el-col span=20>
                            <div style="display: flex; height: 100%; flex: 1;">
                                <el-input v-if="input.type == 'text'" type='textarea' v-model="input.value"></el-input>
                                <el-input v-if="input.type == 'file'" type='file' v-model="input.path"
                                          @change="handleFileChange(index)"></el-input>


                            </div>
                        </el-col>
                        <el-col span=3 align="bottom">
                            <el-button
                                size="small"

                                @click="removeInput(index)">删除输入
                            </el-button>
                        </el-col>

                    </div>


                </el-row>

                <div class="flex-container">
                    <el-button
                        size="small"
                        @click="addText()">添加文本
                    </el-button>

                    <el-button
                        size="small"
                        @click="addImage()">添加图片
                    </el-button>

                    <el-button
                        size="small"
                        type="danger"
                        @click="sendMsg()">发送消息
                    </el-button>
                </div>
            </el-col>


            <div>

            </div>
        </el-row>


    </div>

</template>

<script>
    import {getTags, getContacts, getRooms, sendMsg} from '@/api/getData'

    export default {
        data() {
            return {

                inputs: [{value: '123', type: 'text',}],
                treeData: [
                    {
                        id: 1,
                        label: '一级 1',
                        type: 'tag',
                        children: [
                            {
                                id: 4,
                                label: '二级 1-1',
                                children: [
                                    {
                                        id: 9,
                                        label: '三级 1-1-1'
                                    },
                                    {
                                        id: 10,
                                        label: '三级 1-1-2'
                                    }
                                ]
                            }
                        ]
                    },
                    {
                        id: 2,
                        label: '一级 2',
                        children: [
                            {
                                id: 5,
                                label: '二级 2-1'
                            },
                            {
                                id: 6,
                                label: '二级 2-2'
                            }
                        ]
                    },
                    {
                        id: 3,
                        label: '一级 3',
                        children: [
                            {
                                id: 7,
                                label: '二级 3-1'
                            },
                            {
                                id: 8,
                                label: '二级 3-2'
                            }
                        ]
                    }
                ],
                defaultProps: {
                    children: 'children',
                    label: 'label'
                }
            }
        },


        methods: {
            fileToBase64(file) {
                return new Promise((resolve, reject) => {
                    const reader = new FileReader();
                    reader.onload = () => resolve(reader.result.split(',')[1]);
                    reader.onerror = error => reject(error);
                    reader.readAsDataURL(file);
                });
            },
            handleFileChange(index) {
                let file = null;
                // console.log(event.target.files[0]);
                // console.log(event.currentTarget.files[0]);

                if (event.target === null) {
                    setTimeout(() => {
                        // console.log(event.target.files[0]);
                        file = event.target.files.item(0);
                    }, 2000)
                } else {
                    file = event.target.files[0];
                }

                if (!file) {
                    return;
                }
                this.fileToBase64(file)
                    .then(base64Data => {
                        this.inputs[index].value = base64Data;
                        console.log(base64Data); // 转换后的base64数据
                    })
                    .catch(error => {
                        console.error(error);
                    });

                // 创建文件读取器

                // reader.readAsDataURL(file);
                // // 设置文件读取器完成后的回调
                // reader.onload = (e) => {
                //     const content = e.target.result;
                //     // 在这里处理文件内容
                //     // console.log('content:');
                //     // console.log(content);
                // };

                // 以文本形式读取文件
                reader.readAsText(file);
            },
            removeInput(index) {
                this.inputs.splice(index, 1);

            },
            addImage() {
                this.inputs.push({type: 'file', path: '', value: ''});

            },
            addText() {
                this.inputs.push({value: '', type: 'text'});

            },


            sendMsg() {

                const checkedNodes = this.$refs.tree.getCheckedNodes(true, false);
                console.log(this.inputs)
                let candidates = [];
                for (let i = 0; i < checkedNodes.length; i++) {
                    candidates.push({id: checkedNodes[i].id, name: checkedNodes[i].name, type: checkedNodes[i].type})
                }

                let msgs = [];

                for (let i = 0; i < this.inputs.length; i++) {
                    msgs.push(this.inputs[i])
                }

                sendMsg(candidates, msgs)


            },

            async getTags() {
                const res = await getTags();
                console.log(res)
                console.log(res.data)

                if (res.data != null && res.data.length > 0) {
                    return res.data;
                }
                return null;

            },

            async getContacts(tag) {
                const res = await getContacts(tag);
                if (res.data != null && res.data.length > 0) {
                    return res.data;
                }
                return null;
            },
            async initTree() {
                let treeData = [];
                const tags = await this.getTags();
                console.log('tags : ' + tags);
                console.log(typeof tags);
                for (let i = 0; i < tags.length; i++) {
                    console.log('tags : ' + i + ": " + tags);
                    const contacts = await this.getContacts(tags[i].id);
                    console.log('contacts : ' + JSON.stringify(contacts));
                    let children = [];
                    for (let j = 0; j < contacts.length; j++) {
                        children.push({id: contacts[j].payload.id, label: contacts[j].payload.name, type: 'contact'})
                    }
                    let item = {id: tags[i].id, label: tags[i].id, children: children}
                    treeData.push(item);
                }


                const contactsNotag = await this.getContacts('');
                let children = [];
                for (let j = 0; j < contactsNotag.length; j++) {
                    children.push({
                        id: contactsNotag[j].payload.id,
                        label: contactsNotag[j].payload.name,
                        type: 'contact'
                    })
                }

                treeData.push({id: '-1', label: '无分组', children: children});


                const rooms = await getRooms();
                let roomArray = [];
                for (var i = 0; i < rooms.data.length; i++) {
                    roomArray.push({id: rooms.data[i].payload.id, label: rooms.data[i].payload.topic, type: 'room'});
                }
                treeData.push({id: '-2', label: '群组', children: roomArray});

                this.treeData = treeData;

            }

        },
        created() {
            // 获取标签和联系人列表
            this.initTree();
        }
    }


</script>

<style scoped>

</style>
