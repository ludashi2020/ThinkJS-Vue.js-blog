<template>
  <div>
    <Form ref="formInline" :model="map" inline>
      <div class="search">
        <Button type="success" class="fl" icon="plus" @click="add">添加页面</Button>
        <FormItem>
          <Input type="text" v-model="map.key" placeholder="关键字">
          </Input>
        </FormItem>
        <FormItem>
          <Button type="primary" @click="getList">查询</Button>
        </FormItem>
      </div>
    </Form>
    <Table border :loading="loading" :columns="columns" :data="data.data"></Table>
    <Page class="page" :total="data.count" :page-size="data.pagesize" show-total @on-change="changePage"></Page>
    <Modal v-model="modal" width="360">
      <p slot="header" style="color:#f60;text-align:center">
        <Icon type="ios-information-circle"></Icon>
        <span>删除确认</span>
      </p>
      <div style="text-align:center">
        <p>删除后数据将无法找回，是否继续删除？</p>
      </div>
      <div slot="footer">
        <Button type="error" size="large" long :loading="modal_loading" @click="del">确认删除</Button>
      </div>
    </Modal>
  </div>
</template>
<script>
import { content } from "@/api";
import { Button, Table, Page, Form, FormItem, Input, Modal, Icon } from 'iview';
export default {
  components: {
    Button,
    Table,
    Page,
    Form,
    FormItem,
    Input,
    Modal,
    Icon
  },
  data() {
    return {
      loading: true,
      modal:false,
      modal_loading:false,
      modal_temp:{},
      columns: [{
          title: "页面名称",
          key: "title"
        },
        {
          title: "阅读量",
          key: "view",
          width: 100,
          align: "center"
        },
        {
          title: "发布时间",
          key: "create_time",
          width: 200,
          align: "center",
          render: (h, params) => {
            if (!params.row.create_time) return "";
            return h('span', new Date(params.row.create_time * 1000).toLocaleString());
          }
        },
        {
          title: "操作",
          key: "action",
          width: 150,
          align: "center",
          render: (h, params) => {
            return h("div", [
              h(
                "Button", {
                  props: {
                    type: "primary",
                    size: "small",
                    icon: 'edit'
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.$router.push({
                        path: "/page/save",
                        query: {
                          slug: params.row.slug
                        }
                      });
                    }
                  }
                }
              ),
              h(
                "Button", {
                  props: {
                    type: "error",
                    size: "small",
                    icon: 'trash-a'
                  },
                  on: {
                    click: () => {
                      this.modal=true;
                      this.modal_temp={
                        id:params.row.id,
                        index:params.index
                      }
                    }
                  }
                }
              )
            ]);
          }
        }
      ],
      data: {},
      map: {
        page: 1,
        key: "",
        all: 1,
        pageSize: 10,
        contentType: 'page'
      }
    };
  },
  methods: {
    getList() {
      this.loading = true;
      content.getList(this.map).then(res => {
        this.data = res.data;
        this.loading = false;
      });
    },
    del() {
      if(!this.modal_temp.id){
        return false;
      }
      content.delete(this.modal_temp.id).then(res => {
        this.modal=false;
        if (res.errno == 0){
          this.getList();
          this.$Message.success(res.errmsg);
        }else{
          this.$Message.error(res.errmsg);
        }
      });
    },
    changePage(index) {
      this.map.page = index;
      this.getList();
    },
    add() {
      this.$router.push('/page/save');
    }
  },
  mounted() {
    this.getList();
  }
};

</script>
