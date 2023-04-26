<template>
  <div>
    <crud-table
      title="Lịch sử di dời tàu"
      :tableData="tableData || []"
      :tableColumns="columns"
      @updateData="updateData"
      @exportExcel="exportExcel"
    />
  </div>
</template>
<script>
import CrudTable from "./CrudTable.vue";
import { formatDateTime } from "@/utils";
import { confirmDialogRender } from "@/utils/render";
import { exportRelocationHistory } from "@/api/bm/vesselPlan";
export default {
  components: { CrudTable },
  props: ["berthPlanId"],
  data() {
    return {
      tableData: [],
    };
  },
  computed: {
    columns() {
      return [
        {
          field: "",
          key: "index",
          title: "STT",
          width: 60,
          align: "center",
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return ++rowIndex;
          },
          fixed: "left",
        },
        {
          field: "oldBerthNo",
          key: "oldBerthNo",
          title: "Vị trí cũ",
          width: 160,
          align: "left",
        },
        {
          field: "newBerthNo",
          key: "newBerthNo",
          title: "Vị trí mới",
          width: 160,
          align: "left",
        },
        {
          field: "shiftingDate",
          key: "relocationTime",
          title: "Thời gian di dời",
          width: 160,
          align: "center",
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            if (row.shiftingDate) {
              return formatDateTime(row.shiftingDate)
            }
            return "";
          },
        },
        {
          field: "anchoringTime",
          key: "anchoringTime",
          title: "Thời gian neo bến",
          width: 160,
          align: "right",
        },
        {
          field: "supportedTransport",
          key: "supportedTransport",
          title: "Phương tiện hỗ trợ",
          width: 160,
          align: "left",
        },
      ];
    },
  },
  methods: {
    getHistory() {},
    updateData(key, newData) {
      this.tableData = newData;
    },
    exportExcel() {
      this.$confirm(
        confirmDialogRender(
          this.$MESSAGE_TYPE.WARNING,
          `Xác nhận xuất tất cả dữ liệu di dời tàu?`,
          null
        ),
        "",
        {
          confirmButtonText: "Xuất",
          confirmButtonClass:
            "button-primary button-primary-export message-box-button",
          cancelButtonText: "Hủy bỏ",
          cancelButtonClass:
            "button-secondary button-secondary-default message-box-button",
          center: true,
          dangerouslyUseHTMLString: true,
        }
      )
        .then(() => {
          exportRelocationHistory(this.berthPlanId).then((response) => {
            if (response.code == 200) {
              this.$download.name(response.msg);
            }
            this.$modal.closeLoading();
          });
        })
        .catch(() => {
          this.$modal.closeLoading()
        });
    },
    responseMessage(msg, type) {
      this.$confirm(confirmDialogRender(type, msg, null), "", {
        cancelButtonText: "Đồng ý",
        cancelButtonClass:
          "button-primary button-primary-default message-box-button",
        showConfirmButton: false,
        center: true,
        showClose: false,
        dangerouslyUseHTMLString: true,
      }).catch(() => {
        this.isSearch = !this.isSearch;
      });
    },
  },
};
</script>
<style lang="">
</style>