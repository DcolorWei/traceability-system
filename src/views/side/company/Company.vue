<template>
  <n-data-table :columns="columns" :data="companydata" :bordered="false" />
  <div class="affix" @click="addCompany()">
    <n-icon size="40" :component="Add" />
  </div>
</template>

<script lang="ts">
import { Add } from "@vicons/ionicons5";
import { defineComponent, h, reactive, ref } from "vue";
import { NDataTable, DataTableColumns, NButton, NIcon } from "naive-ui";
import { Company } from "./Company.entity";
import axios from "axios";
axios.defaults.withCredentials = true;

const createColumns = ({
  deleteCompany,
}: {
  deleteCompany: (row: Company) => void;
}): DataTableColumns<Company> => {
  return [
    {
      title: "企业ID",
      key: "companyId",
      align: "center",
    },
    {
      title: "企业名称",
      key: "companyName",
      align: "center",
    },
    {
      title: "企业类型",
      key: "companyType",
      align: "center",
    },
    {
      title: "操作",
      key: "actions",
      align: "center",
      width: 300,
      render(row) {
        return [
          h(
            NButton,
            {
              tertiary: true,
              size: "small",
              style: "margin:1%;background:#EAEAEA",
            },
            { default: () => "查看" }
          ),
          h(
            NButton,
            {
              tertiary: true,
              size: "small",
              style: "margin:1%;background:#FAC11B",
            },
            { default: () => "发信" }
          ),
          h(
            NButton,
            {
              tertiary: true,
              size: "small",
              style: "margin:1%;background:#E63F32",
              onclick: () => deleteCompany(row),
            },
            { default: () => "删除" }
          ),
        ];
      },
    },
  ];
};

let companydata: Company[] = reactive([]);
const formValue = ref({
  user: {
    name: "",
    age: "",
  },
  phone: "",
});
axios({
  url: "https://cunyuqing.online:8081/company/getJointVenture",
  method: "GET",
}).then((res) => {
  //清空
  while (companydata.length > 0) {
    companydata.shift();
  }

  if (res != null) {
    res.data.forEach((element: Company) => {
      companydata.push(element);
    });
  }
});

function addCompany() {
  let companyId = prompt("输入对方ID");
  axios({
    url: "https://cunyuqing.online:8081/company/makeFriends",
    method: "POST",
    data: {
      companyId: Number(companyId),
    },
  })
    .then(() => {
      alert("发送请求成功");
    })
    .catch(() => {
      alert("请求重复");
    });
}

export default defineComponent({
  setup() {
    return {
      companydata,
      columns: createColumns({
        deleteCompany(row: Company) {
          if (confirm(`确定要删除与${row.companyName}联系？`)) {
            axios({
              url: "https://cunyuqing.online:8081/company/deleteFriends",
              method: "POST",
              data: {
                companyId: row.companyId,
              },
            })
              .then(() => {
                axios({
                  url: "https://cunyuqing.online:8081/company/getJointVenture",
                  method: "GET",
                }).then((res) => {
                  //清空
                  while (companydata.length > 0) {
                    companydata.shift();
                  }

                  if (res != null) {
                    res.data.forEach((element: Company) => {
                      companydata.push(element);
                    });
                  }
                });
                alert("删除成功！");
              })
              .catch((err) => {
                alert(err.message);
              });
          }
        },
      }),
      addCompany,
      Add,
      formValue,
    };
  },
  components: {
    NDataTable,
    NIcon,
  },
});
</script>

<style scoped>
.affix {
  width: 40px;
  height: 40px;
  background: #ffd664;
  border: 3px solid #474747;
  border-radius: 50%;
  position: fixed;
  right: 6%;
  bottom: 20%;
}

.form-table {
  padding: 1% 5%;
  background: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.btns {
  display: flex;
  justify-content: space-around;
}
</style>