<template>
  <div class="main-view__body">
    <div class="one-panel main-view__body-content">
      <div class="body-content-top">
        <div class="shift-name">
          <div>
            <span>Ngày: </span>
            <span class="text-value">{{ currDate }}</span>
          </div>
          <div>
            <span>Ca: </span>
            <span class="text-value">{{ currShift }}</span>
          </div>
        </div>
      </div>
      <div class="one-panel main-view__body-content main-view__vessel-plan">
        <ve-table
          id="loading-container"
          :columns="columns"
          :table-data="berths"
          row-key-field-name="id"
          :scroll-width="'100%'"
          :border-around="true"
          :border-x="true"
          :border-y="true"
          :row-style-option="rowStyleOption"
          :editOption="editOption"
          :column-width-resize-option="columnWidthResizeOption"
          :cell-style-option="cellStyleOption"
        />
      </div>
      <relocation-dialog
        @close-form="closeRelocation"
        :relocationForm="relocationForm"
        @reset-grid="resetGrid"
      />
      <update-time-dialog
        @close-form="closeUpdateTime"
        :updateTimeForm="updateTimeForm"
        :isForecast="isForecast"
        @reset-grid="resetGrid"
      />
      <detail-dialog @reset-grid="resetGrid" />
      <diary-dialog @close-form="closeDiary" :diaryForm="diaryForm" @reset-grid="resetGrid" />
    </div>
  </div>
</template>

<script>
import { listVesselPlanDocked, updateVesselPlan } from "@/api/bm/vesselPlan";
import { checkPermi } from "@/utils/permission";
import moment from "moment";
import { listBerthNotPage } from "@/api/bm/berthList";
import RelocationDialog from "./RelocationDialog.vue";
import UpdateTimeDialog from "../../components/UpdateTimeDialog";
import DetailDialog from "./DetailDialog.vue";
import DiaryDialog from "./DiaryDialog.vue";
import { formatDateTime } from "@/utils";
import { confirmDialogRender } from "@/utils/render";
const shiftTime = [
  {
    code: "K1",
    from: "00:00:00",
    to: "05:59:59",
    label: "Ca 1 (00h00 - 06h00)",
  },
  {
    code: "K2",
    from: "06:00:00",
    to: "11:59:59",
    label: "Ca 2 (06h00 - 12h00)",
  },
  {
    code: "K3",
    from: "12:00:00",
    to: "17:59:59",
    label: "Ca 3 (12h00 - 18h00)",
  },
  {
    code: "K4",
    from: "18:00:00",
    to: "23:59:59",
    label: "Ca 4 (18h00 - 24h00)",
  },
];
export default {
  dicts: ["bm_vessl_plan_status"],
  props: ["queryParams", "isSearch"],
  components: { RelocationDialog, UpdateTimeDialog, DetailDialog, DiaryDialog },
  data() {
    return {
      isForecast: true,
      queryBerths: {
        isAsc: "asc",
        orderByColumn: "fromMeter",
      },
      relocationForm: {},
      updateTimeForm: {},
      diaryForm: {},
      loadingInstance: null,
      defaultProps: {
        children: "children",
        label: "label",
      },
      rowStyleOption: {
        hoverHighlight: false,
      },
      cellSelectionOption: {
        // disble cell selection
        enable: true,
      },
      columnWidthResizeOption: {
        // default false
        enable: true,
        // column resize min width
        minWidth: 60,
      },
      // edit option
      editOption: {
        // cell value change
        cellValueChange: ({ row, column }) => {},
      },
      // sortOption: {
      //   sortChange: (params) => {
      //     this.sort(params);
      //     this.getList();
      //   },
      // },
      berthList: [],
      berths: [],
      columns: [
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
          field: "berthNo",
          key: "berthNo",
          width: 100,
          title: "Bến",
          align: "left",
          // // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          edit: true,
          fixed: "left",
        },
        {
          field: "vesselName",
          key: "vesselName",
          width: 260,
          title: "Tên tàu",
          align: "left",
          fixed: "left",
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            // return this.dataTransfer[row.berthNo]?.vessel?.vesselName
            const vessel = this.dataTransfer[row.berthNo]?.vessel?.[row.index];
            if (!vessel) return "";
            return (
              <div class="grid-cell-button">
                <span
                  title={vessel?.vesselName}
                  class="link-text"
                  onClick={() => {
                    this.$router.push({ query: { id: vessel.id } });
                  }}
                >
                  {vessel?.vesselName}
                </span>
                <div class="button-style">
                  {checkPermi(["bm:berth-plan:plan-time"]) ||
                  checkPermi(["bm:berth-plan:actual-time"]) ? (
                    <div
                      class="cell-button"
                      title="Cập nhập thời gian"
                      onClick={() => this.openUpdateTime(vessel)}
                    >
                      <svg-icon
                        icon-class="green-calendar"
                        width="12px"
                        height="13px"
                      />
                    </div>
                  ) : (
                    ""
                  )}
                  {checkPermi(["bm:vessel-diary:add"]) ? (
                    <div
                      class="cell-button fix-button"
                      title="Nhật ký khai thác"
                      style="cursor: pointer;"
                      onClick={() => {
                        if (!vessel.atb) {
                          this.$emit(
                            "response-message",
                            "Tàu chưa cập bến!",
                            this.$MESSAGE_TYPE.ERROR
                          );
                        } else {
                          this.openVesselDiary(vessel);
                        }
                      }}
                    >
                      <svg-icon icon-class="reload"></svg-icon>
                    </div>
                  ) : (
                    <div></div>
                  )}
                  {checkPermi(["bm:berth-plan:docked"]) ? (
                    <div
                      class="cell-button"
                      title="Huỷ lệnh cập"
                      onClick={() => this.clearDocked(vessel)}
                    >
                      <svg-icon icon-class="clear" width="12px" height="13px" />
                    </div>
                  ) : (
                    ""
                  )}
                  {checkPermi(["bm:berth-shifting:add"]) ? (
                    <div
                      class="cell-button fix-button"
                      title="Di dời tàu"
                      style="cursor: pointer;"
                      onClick={() => {
                        if (!vessel.atb) {
                          this.$emit(
                            "response-message",
                            "Tàu chưa cập bến!",
                            this.$MESSAGE_TYPE.ERROR
                          );
                        } else {
                          this.openRelocation(vessel);
                        }
                      }}
                    >
                      <svg-icon icon-class="blue-ship"></svg-icon>
                    </div>
                  ) : (
                    ""
                  )}
                  {checkPermi(["bm:berth-plan:export"]) ? (
                    <div
                      class="cell-button"
                      title="Xuất excel"
                      onClick={() => this.exportDocked(vessel.vesselId, vessel.callSeq, vessel.callYear)}
                    >
                      <svg-icon icon-class="default-download" width="12px" height="13px" />
                    </div>
                  ) : (
                    ""
                  )}
                </div>
              </div>
            );
          },
        },
        {
          field: "etb",
          key: "etb",
          width: 140,
          title: "Ngày cập",
          align: "center",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            if (vessel?.[row.index]?.atb) {
              return (
                <span style="color: #FFB020;">
                  {formatDateTime(vessel?.[row.index].atb)}
                </span>
              );
            } else if (vessel?.[row.index]?.etb) {
              return formatDateTime(vessel?.[row.index].etb);
            }
            return "";
          },
        },
        {
          field: "etd",
          key: "etd",
          width: 140,
          title: "Ngày dự kiến rời",
          align: "center",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            if (vessel?.[row.index]?.etd) {
              return formatDateTime(vessel?.[row.index].etd);
            }
            return "";
          },
        },
        {
          field: "consignee",
          key: "consignee",
          width: 280,
          title: "Chủ hàng",
          align: "left",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.consignee;
          },
        },
        {
          field: "cargoType",
          key: "cargoType",
          width: 160,
          title: "Loại hàng",
          align: "left",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.cargoType;
          },
        },
        {
          field: "unit",
          key: "unit",
          width: 80,
          title: "ĐVT",
          align: "center",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.unit;
          },
        },
        {
          field: "amount",
          key: "amount",
          width: 120,
          title: "Số lượng",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.amount;
          },
        },
        {
          field: "loaded",
          key: "loaded",
          width: 120,
          title: "Đã xếp dỡ",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.loaded;
          },
        },
        {
          field: "remain",
          key: "remain",
          width: 120,
          title: "Còn lại",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            if (vessel?.[row.index]?.amount) {
              return vessel?.[row.index]?.amount - vessel?.[row.index]?.loaded;
            }
            return "";
          },
        },
        {
          field: "workerGroup",
          key: "workerGroup",
          width: 180,
          title: "Tổ công nhân",
          align: "left",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.workerGroup;
          },
        },
        {
          field: "equipment",
          key: "equipment",
          width: 180,
          title: "Phương tiện",
          align: "left",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.equipment;
          },
        },
        {
          field: "remark",
          key: "remark",
          width: 140,
          title: "Ghi chú",
          align: "left",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            const vessel = this.dataTransfer[row.berthNo]?.vessel;
            return vessel?.[row.index]?.remark;
          },
        },
      ],
      // Berth table data
      vesselPlanList: [],
      currShift: "",
      currDate: "",
      cellStyleOption: {
        bodyCellClass: ({ row, column, rowIndex }) => {
          if (!this.dataTransfer[row.berthNo].isShow) {
            return "hidden-cell";
          }
        },
      },
    };
  },
  watch: {
    isSearch(newValue) {
      this.getList();
    },
  },
  mounted() {
    this.loadingInstance = this.$veLoading({
      target: document.querySelector("#loading-container"),
      name: "wave",
    });
    this.getList();
    this.getCurrShift();
    this.getBerthList();
  },
  computed: {
    dataTransfer() {
      const result = {};
      this.berthList.map((val) => {
        result[val.berthNo] = { ...val };

        if (val.berthNo === "PHAO") {
          result[val.berthNo] = { ...val, isShow: true };
          return;
        }
        // const parent = this.findBerthParent(val);
        // if (!parent) {
        result[val.berthNo] = {
          ...result[val.berthNo],
          ...val,
          isShow: true,
        };
        //   return;
        // }
        // result[val.berthNo] = { ...val, parent: parent.berthNo, isShow: true };
        // result[parent.berthNo] = {
        //   ...result[parent.berthNo],
        //   children: [...(result[parent.berthNo]?.children || []), val.berthNo],
        // };
      });
      this.vesselPlanList.map((val) => {
        if (!val.berthNo) {
          return;
        }
        // result[val.berthNo].vessel = { ...val };
        if (!result[val.berthNo].vessel) {
          result[val.berthNo].vessel = [val];
          return;
        }
        result[val.berthNo].vessel.map((curr) => {
          if (val.fromPos - curr.toPos >= 0 || 0 <= curr.fromPos - val.toPos) {
            result[val.berthNo].vessel = [...result[val.berthNo].vessel, val];
          }
        });
        // if (result[val.berthNo].parent) {
        //   result[val.berthNo].parent;
        //   const parentBerth = result[val.berthNo].parent;
        //   result[parentBerth].isShow = false;
        //   return;
        // }
        // result[val.berthNo]?.children?.map((item) => {
        //   result[item].isShow = false;
        // });
      });
      const berthList = [];
      this.berthList.map((berth) => {
        if (!result[berth.berthNo].vessel?.length) {
          berthList.push({ ...berth, id: berth.id });
          return;
        }
        result[berth.berthNo].vessel.map((vessel, index) => {
          berthList.push({ ...berth, index, id: berth.id.toString() + index });
        });
      });
      this.berths = berthList;
      return result;
    },
  },
  methods: {
    findBerthParent(berth) {
      const result = this.berthList.find(
        (val) =>
          val.berthNo !== berth.berthNo &&
          val.fromMeter <= berth.fromMeter &&
          val.toMeter >= berth.toMeter
      );
      return result;
    },
    sort(params) {},
    /** Query berth list */
    getList() {
      this.loadingInstance.show();
      listVesselPlanDocked(this.queryParams)
        .then((response) => {
          this.vesselPlanList = response.rows;
          this.loadingInstance.close();
        })
        .catch((error) => {
          this.loadingInstance.close();
        });
    },
    editVesselPlan(id) {
      this.$emit("edit-vessel-plan", id);
    },
    formatWeightNumber(number) {
      let numberStr = number.toString();
      return numberStr.replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    getCurrShift() {
      this.currDate = moment().format("DD/MM/YYYY");
      for (let item in shiftTime) {
        if (
          moment().isBetween(
            moment(shiftTime[item].from, "HH:mm:ss"),
            moment(shiftTime[item].to, "HH:mm:ss"),
            undefined,
            "[]"
          )
        ) {
          this.currShift = shiftTime[item].label;
        }
        continue;
      }
    },
    getBerthList() {
      listBerthNotPage(this.queryBerths).then((res) => {
        if (res.code == 200) {
          this.berthList = [...res.data];
          this.berths = [...res.data];
        }
      });
    },
    openRelocation(vessel) {
      // if (vessel.atb) {
      //    // to do : mở form di dời tàu
      //    this.relocationForm = {open: true, title: 'Di dời tàu', vessel}
      // } else {
      //   this.$emit("response-message", "Vui lòng cập nhật ngày cập tàu!", this.$MESSAGE_TYPE.WARNING);
      // }
      this.relocationForm = { open: true, title: "Di dời tàu", vessel };
    },
    openUpdateTime(vessel) {
      // to do : mở form cập nhập thời gian
      this.updateTimeForm = { open: true, title: "Cập nhật thời gian", vessel };
    },
    openVesselDiary(vessel) {
      // to do : mở form nhật ký khai thác
      this.diaryForm = { open: true, title: "Nhật ký khai thác", vessel };
    },
    closeRelocation() {
      this.relocationForm = { open: false, title: "Di dời tàu" };
      // to do : mở form di dời tàu
    },
    closeUpdateTime() {
      // to do : mở form cập nhập thời gian
      this.updateTimeForm = { open: false, title: "Cập nhật thời gian" };
    },
    closeDiary() {
      // to do : đóng form nhật ký khai thác
      this.diaryForm = { open: false, title: "Nhật ký khai thác" };
    },
    resetGrid() {
      this.getList();
    },
    clearDocked(vessel) {
      if (vessel.status == "1") {
        this.$confirm(
          confirmDialogRender(
            this.$MESSAGE_TYPE.WARNING,
            `Xác nhận huỷ lệnh cập của tàu ` + vessel.vesselName + `!`,
            null
          ),
          "",
          {
            confirmButtonText: "Xác nhận",
            confirmButtonClass:
              "button-primary button-primary-delete message-box-button",
            cancelButtonText: "Hủy bỏ",
            cancelButtonClass:
              "button-secondary button-secondary-default message-box-button",
            center: true,
            dangerouslyUseHTMLString: true,
          }
        ).then(() => {
          vessel.listBMBerthPlanOther = [];
          vessel.listBMBerthShiftings = [];
          vessel.status = "0";
          updateVesselPlan(vessel).then((response) => {
            if (response.code == 200) {
              this.$emit(
                "response-message",
                "Huỷ lệnh cập tàu thành công!",
                this.$MESSAGE_TYPE.SUCCESS
              );
            } else {
              this.$emit(
                "response-message",
                "Huỷ lệnh cập tàu thất bại!",
                this.$MESSAGE_TYPE.ERROR
              );
            }
          });
        });
      } else {
        this.$emit(
          "response-message",
          "Không thể huỷ lệnh cập, tàu đã cập bến!",
          this.$MESSAGE_TYPE.ERROR
        );
      }
    },
    exportDocked(vesselId, callSeq, callYear) {
      const berthPlanCode = `${vesselId}${callSeq}${callYear}`;
      this.$emit("export-docked", berthPlanCode);
    },
  },
};
</script>
<style lang="scss">
@import "@/assets/styles/base-variables.scss";
.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  display: inline-block;
  margin-right: 8px;
}
.vessel_plan__status {
  display: flex;
  align-items: center;
  &.status_0 {
    color: $neutral-n8;

    .dot {
      background-color: $neutral-n5;
    }
  }
  &.status_1 {
    color: $purple-p5;

    .dot {
      background-color: $purple-p5;
    }
  }
  &.status_2 {
    color: $blue-b5;
    .dot {
      background-color: $blue-b5;
    }
  }
  &.status_3 {
    color: $yellow-y5;
    .dot {
      background-color: $yellow-y5;
    }
  }
  &.status_4 {
    color: $green-g5;
    .dot {
      background-color: $green-g5;
    }
  }
}
.main-view__vessel-plan {
  height: calc(100% - 44px);
}
.shift-name {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  font-weight: 400;
  font-size: 14px;
  line-height: 20px;
  color: $neutral-n7;
  background-color: $neutral-n0;
  border-radius: 8px 8px 0px 0px;
  .text-value {
    font-weight: 400;
    font-size: 14px;
    line-height: 20px;
    color: $red-r5;
    margin-left: 12px;
  }
}
.ve-table-body-tr:has(.hidden-cell) {
  display: none;
}
</style>
