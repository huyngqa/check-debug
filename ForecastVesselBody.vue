<template>
  <div class="main-view__body">
    <div class="one-panel main-view__body-content">
      <div class="body-top__container">
        <div class="body-content-top">
          <div>
            <span class="select-label">Loại Tàu</span>
            <el-select
              v-model="arrayOfvesselType2"
              class="el-select-body-content-top"
              multiple
              collapse-tags
              placeholder="Chọn"
              @change="changeVesselType2Input"
            >
              <el-option
                v-for="item in listVesseType2"
                :key="item"
                :label="item"
                :value="item"
              >
              </el-option>
            </el-select>
          </div>

          <div>
            <span class="select-label">Loại Hàng</span>
            <el-select
              v-model="arayOfCargos"
              class="el-select-body-content-top"
              multiple
              collapse-tags
              placeholder="Chọn"
              @change="changeCargosInput"
            >
              <el-option
                v-for="item in listCargos"
                :key="item.name"
                :label="item"
                :value="item"
              >
              </el-option>
            </el-select>
          </div>
          <div class="date-picker__container">
            <div>
              <el-date-picker
                v-model="queryParams.params.beginTime"
                value-format="yyyy-MM-dd"
                format="dd/MM/yyyy"
                type="date"
                placeholder="Từ ngày"
                class="input-date"
                @change="search"
              ></el-date-picker>
            </div>
            <span class="date-picker-line" />
            <div>
              <el-date-picker
                v-model="queryParams.params.endTime"
                value-format="yyyy-MM-dd"
                format="dd/MM/yyyy"
                type="date"
                placeholder="Đến ngày"
                class="input-date"
                @change="search"
              ></el-date-picker>
            </div>
          </div>
        </div>
      </div>

      <div class="body-content-bottom table-form table__view-edit">
        <ve-table
          id="loading-container"
          :columns="columns"
          :table-data="forecastVessels"
          :checkbox-option="checkboxOption"
          row-key-field-name="id"
          :scroll-width="'100%'"
          :sort-option="sortOption"
          :border-around="true"
          :border-x="true"
          :border-y="true"
          :row-style-option="rowStyleOption"
          :cellSelectionOption="cellSelectionOption"
          :editOption="editOption"
          :column-width-resize-option="columnWidthResizeOption"
          :cell-autofill-option="cellAutofillOption"
          :cell-style-option="cellStyleOption"
        />
      </div>
      <update-time-dialog
        @close-form="closeUpdateTime"
        :updateTimeForm="updateTimeForm"
        @reset-grid="resetGrid"
      />
    </div>
  </div>
</template>

<script>
import { listForecastVessel, getInfoByBerthPlanId } from "@/api/bm/vesselPlan";
import { checkPermi } from "@/utils/permission";
import { listBerthNotPage } from "../../../api/bm/berthList";
import { validate } from "@/utils/validateData";
import UpdateTimeDialog from "./UpdateTimeDialog";
import { getFastCargoList } from "@/api/masterDataApi/masterDataApi";
import moment from "moment";

export default {
  dicts: ["bm_vessl_plan_status"],
  props: ["queryParams", "isSearch", "type"],
  components: {
    UpdateTimeDialog,
  },
  data() {
    return {
      updateTimeForm: {},
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
      listVesseType2: [
        "Multi-purpose",
        "Bulk carrier",
        "RO/RO vessel",
        "Barge",
        "Passenger ship",
        "Container",
      ],
      // edit option
      editOption: {
        //after cell value change
        afterCellValueChange: ({ row, column, changeValue }) => {
          const index = this.forecastVessels.findIndex(
            (val) => val.id === row.id
          );
          if (column.field == "remark") {
            this.checkRowChange(index);
            return;
          }
          const data = [...this.forecastVessels];
          // chỉ cho edit những trường số lượng nên cần parse sang type number
          data[index] = {
            ...this.forecastVessels[index],
            [column.field]: !isNaN(parseFloat(changeValue))
              ? parseFloat(changeValue)
              : null,
          };
          this.forecastVessels = [...data];
          this.checkRowChange(index);
        },
      },
      columnWidthResizeOption: {
        // default false
        enable: true,
        // column resize min width
        minWidth: 60,
      },
      checkboxOption: {
        // row select change event
        selectedRowChange: ({ row, isSelected, selectedRowKeys }) => {
          this.changeSelectedRowKeys(selectedRowKeys);
        },
        // selected all change event
        selectedAllChange: ({ isSelected, selectedRowKeys }) => {
          this.changeSelectedRowKeys(selectedRowKeys);
        },
      },
      autoFillCols: [3, 7, 8, 9, 10, 15, 16, 17],
      cellAutofillOption: {
        directionX: false,
        directionY: true,
        beforeAutofill: ({ sourceSelectionRangeIndexes }) => {
          if (
            !this.autoFillCols.includes(
              sourceSelectionRangeIndexes.startColIndex
            )
          ) {
            return false;
          }
        },
        afterAutofill: ({
          direction,
          sourceSelectionRangeIndexes,
          targetSelectionRangeIndexes,
        }) => {
          if (direction === "down") {
            for (
              let i = sourceSelectionRangeIndexes.startRowIndex + 1;
              i <= targetSelectionRangeIndexes.endRowIndex;
              i++
            ) {
              this.checkRowChange(i);
            }
            return;
          }
          for (
            let i = targetSelectionRangeIndexes.startRowIndex;
            i < sourceSelectionRangeIndexes.endRowIndex;
            i++
          ) {
            this.checkRowChange(i);
          }
        },
      },
      sortOption: {
        sortChange: (params) => {
          this.sort(params);
          this.getList();
        },
      },
      rowsChange: {},
      cellStyleOption: {
        bodyCellClass: ({ row, column, rowIndex }) => {
          if (this.rowsChange[row.id]) {
            return "changed-row";
          }
          if (row.vesselType2 == "Container") {
            return "blue-row";
          }
        },
      },
      rootForecastVessels: [],
      columns: [
        {
          field: "",
          key: "cb",
          type: "checkbox",
          title: "",
          width: 50,
          align: "center",
          fixed: "left",
        },
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
          field: "vesselName",
          key: "vesselName",
          width: 240,
          title: "Tên tàu",
          align: "left",
          fixed: "left",
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return (
              <div class="grid-cell-button">
                <span
                  title={row.vesselName}
                  class="link-text"
                  on-click={() =>
                    checkPermi(["bm:berth-plan:query"])
                      ? this.editForecastVessel(row.id, row.vesselId)
                      : ""
                  }
                >
                  {row.vesselName}
                </span>
                <div class="button-style">
                  {checkPermi(["bm:berth-plan:plan-time"]) ? (
                    <div
                      class="cell-button"
                      title="Cập nhập thời gian"
                      onClick={() => this.openUpdateTime(row)}
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
                </div>
              </div>
            );
          },
        },
        {
          field: "berthNo",
          key: "berthNo",
          width: 140,
          title: "Bến",
          align: "left",
          // sortBy: "",
          // ellipsis: {
          //   showTitle: true,
          // },
          // // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return (
              <el-select
                v-model={this.forecastVessels[rowIndex].berthNo}
                placeholder=""
                class="input-temp"
                style="width: 100%"
                onChange={(e) => {
                  this.checkRowChange(rowIndex);
                }}
              >
                {this.berthList?.map((item) => (
                  <el-option
                    key={item.id}
                    label={item.berthNo}
                    value={item.berthNo}
                  ></el-option>
                ))}
              </el-select>
            );
          },
        },
        // {
        //   field: "vesselId",
        //   key: "vesselId",
        //   width: 200,
        //   title: "Tàu-Năm-Chuyến",
        //   align: "center",
        //   sortBy: "",
        //   ellipsis: {
        //     showTitle: true,
        //   },
        //   edit: true,
        // },
        // {
        //   key: "status",
        //   field: "status",
        //   title: "Trạng Thái",
        //   width: 140,
        //   align: "left",
        //   // sortBy: "",
        //   renderBodyCell: ({ row, column, rowIndex }, h) => {
        //     const status = this.dict.type.bm_vessl_plan_status.find(
        //       (val) => val.value === row.status
        //     );
        //     return (
        //       <div class={`vessel_plan__status status_${row.status}`}>
        //         <div class="dot" />
        //         <span>{status?.label}</span>
        //       </div>
        //     );
        //   },
        // },
        // {
        //   field: "vesselType2",
        //   key: "vesselType2",
        //   title: "Loại tàu 2",
        //   width: 140,
        //   align: "left",
        //   ellipsis: {
        //     showTitle: true,
        //   },
        // },
        {
          field: "yardClose",
          key: "yardClose",
          width: 200,
          title: "Closing time",
          align: "center",
          // sortBy: "",
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return (
              <el-date-picker
                style={{ width: "100%" }}
                v-model={this.forecastVessels[rowIndex].yardClose}
                format="dd/MM/yyyy HH:mm"
                value-format="yyyy-MM-dd HH:mm"
                type="datetime"
                onChange={() => {
                  this.checkRowChange(rowIndex);
                }}
              ></el-date-picker>
            );
          },
        },
        {
          field: "eta",
          key: "eta",
          width: 200,
          title: "Giờ đến",
          align: "center",
          // sortBy: "asc",
          // ellipsis: {
          //   showTitle: true,
          // },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return (
              <el-date-picker
                style={{ width: "100%" }}
                v-model={this.forecastVessels[rowIndex].eta}
                format="dd/MM/yyyy HH:mm"
                value-format="yyyy-MM-dd HH:mm"
                type="datetime"
                onChange={() => {
                  this.checkRowChange(rowIndex);
                }}
              ></el-date-picker>
            );
          },
        },
        {
          field: "etb",
          key: "etb",
          width: 200,
          title: "Giờ cập",
          align: "center",
          // sortBy: "",
          // ellipsis: {
          //   showTitle: true,
          // },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return (
              <el-date-picker
                style={{ width: "100%" }}
                v-model={this.forecastVessels[rowIndex].etb}
                format="dd/MM/yyyy HH:mm"
                value-format="yyyy-MM-dd HH:mm"
                type="datetime"
                onChange={async (val) => {
                  let totalNormTime = 0;
                  await getInfoByBerthPlanId(row.id).then((response) => {
                    if (response.data) {
                      response.data.forEach((obj) => {
                        totalNormTime += obj.normTime;
                      });
                    }
                  });
                  const totalHours = parseFloat(totalNormTime);
                  if (isNaN(totalHours)) return;
                  const etbDate = new Date(val);
                  const etdDate = moment(etbDate).add(totalHours, "hours");
                  this.forecastVessels[rowIndex].etd = etdDate.format("YYYY-MM-DD HH:mm");
                  this.checkRowChange(rowIndex);
                }}
              ></el-date-picker>
            );
          },
        },
        {
          field: "etd",
          key: "etd",
          width: 200,
          title: "Giờ rời",
          align: "center",
          // sortBy: "",
          // ellipsis: {
          //   showTitle: true,
          // },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return (
              <el-date-picker
                style={{ width: "100%" }}
                v-model={this.forecastVessels[rowIndex].etd}
                format="dd/MM/yyyy HH:mm"
                value-format="yyyy-MM-dd HH:mm"
                type="datetime"
                onChange={() => {
                  this.checkRowChange(rowIndex);
                }}
              ></el-date-picker>
            );
          },
        },
        {
          field: "draft",
          key: "draft",
          width: 120,
          title: "Mớn nước",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          // edit: true,
        },
        {
          field: "loa",
          key: "loa",
          width: 120,
          title: "LOA(m)",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          // edit: true,
        },
        {
          field: "cargoType",
          key: "cargoType",
          width: 120,
          title: "Loại hàng",
          align: "left",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            if (row.vesselType2 == "Container" && row.cargoType) {
              return "CONTAINER";
            }
            return row.cargoType;
          },
        },
        {
          field: "unit",
          key: "unit",
          width: 100,
          title: "ĐVT",
          align: "center",
          // sortBy: "",
          // ellipsis: {
          //   showTitle: true,
          // },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            if (!row.cargoType) {
              return "";
            }
            return row.unit;
          },
        },
        {
          field: "import_weight",
          key: "import_weight",
          width: 150,
          title: "Số lượng nhập",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return row.import_weight
              ? this.formatWeightNumber(row.import_weight)
              : "";
          },
        },
        {
          field: "export_weight",
          key: "export_weight",
          width: 150,
          title: "Số lượng xuất",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return row.export_weight
              ? this.formatWeightNumber(row.export_weight)
              : "";
          },
        },
        {
          field: "cargoWeight",
          key: "cargoWeight",
          width: 140,
          title: "Tổng cộng",
          align: "right",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          // edit: true,
          renderBodyCell: ({ row, column, rowIndex }, h) => {
            return this.formatWeightNumber(
              parseFloat(this.forecastVessels[rowIndex].import_weight || 0) +
                parseFloat(this.forecastVessels[rowIndex].export_weight || 0)
            );
          },
        },
        {
          field: "citizenshipName",
          key: "citizenshipName",
          title: "Quốc tịch",
          width: 160,
          align: "left",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          // edit: true,
        },
        {
          field: "vesselType2",
          key: "vesselType2",
          width: 140,
          title: "Loại tàu",
          align: "left",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
        },
        {
          field: "remark",
          key: "remark",
          width: 200,
          title: "Ghi chú",
          align: "left",
          // sortBy: "",
          ellipsis: {
            showTitle: true,
          },
          edit: true,
        },
      ],
      berthList: [],
      // Berth table data
      forecastVessels: [],
      sotedForecastVessels: [],
      sotedCargos: [],
      arrayOfvesselType2: [],
      listCargos: [],
      arayOfCargos: [],
    };
  },
  watch: {
    isSearch(newValue) {
      this.getList();
    },
  },
  async created() {
    listBerthNotPage().then((res) => {
      if (res.code == 200) {
        this.berthList = res.data;
      }
    });
    await getFastCargoList({}).then((response) => {
      this.listCargos = response.data?.rows?.map((obj) => obj.name.trim());
    });
  },
  async mounted() {
    this.loadingInstance = this.$veLoading({
      target: document.querySelector("#loading-container"),
      name: "wave",
    });
    this.getList();
  },
  methods: {
    sort(params) {
      if ("asc" === params.eta || "desc" === params.eta) {
        this.queryParams.orderByColumn = "eta";
        this.queryParams.isAsc = params.eta;
        this.$emit("search", { orderByColumn: key });
        return;
      }
    },
    /** Query berth list */
    getList() {
      this.loadingInstance.show();
      listForecastVessel(this.queryParams)
        .then((response) => {
          if (response.code == 200) {
            response.rows?.forEach((element) => {
              if (element.remark == null) {
                element.remark = "";
              }
            });
            this.forecastVessels = [...response.rows];
            this.sotedForecastVessels = [...response.rows];
            this.sotedCargos = [...response.rows];
            this.rootForecastVessels = _.cloneDeep(response.rows);
            this.$emit("get-data", response);
            this.loadingInstance.close();
          }
        })
        .catch((error) => {
          this.loadingInstance.close();
        });
    },
    changeSelectedRowKeys(keys) {
      this.$emit("select-handler", keys);
    },
    editForecastVessel(id, vesselId) {
      this.$emit("edit-forecast-vessel", id, vesselId);
    },
    formatWeightNumber(number) {
      let numberStr = number.toString();
      return numberStr.replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    search() {
      this.$emit("search", this.queryParams);
    },
    openUpdateTime(vessel) {
      // to do : mở form cập nhập thời gian
      this.updateTimeForm = {
        open: true,
        title: "Cập nhật thời gian",
        vessel: JSON.parse(JSON.stringify(vessel)),
      };
    },
    closeUpdateTime() {
      // to do : mở form cập nhập thời gian
      this.updateTimeForm = { open: false, title: "Cập nhật thời gian" };
    },
    checkRowChange(index) {
      const isChanged = _.isEqual(
        this.rootForecastVessels[index],
        this.forecastVessels[index]
      );
      this.rowsChange = {
        ...this.rowsChange,
        [this.rootForecastVessels[index].id]: !isChanged,
      };
    },
    resetRowsChanged() {
      this.rowsChange = {};
    },
    validateData(val) {
      return validate(val)
        .isArray(val)
        .children({
          eta: (val) => {
            return validate(val).isRequired("Giờ đến không được phép trống!");
          },
          etb: (val, item) => {
            if (item.eta) {
              return validate(val).compareDate(
                item.eta,
                val,
                'Giờ cập phải lớn hơn hoặc bằng giờ đến, ở dự báo tàu "' +
                  item.vesselName +
                  '" !',
                "same"
              );
            }
            return validate(val);
          },
          etd: (val, item) => {
            if (item.etb) {
              return validate(val).compareDate(
                item.etb,
                val,
                'Giờ rời phải lớn hơn hoặc bằng giờ cập, ở dự báo tàu "' +
                  item.vesselName +
                  '" !',
                "same"
              );
            }
            return validate(val);
          },
          import_weight: (val) => {
            if (!val) {
              return validate(val);
            }
            return validate(val)
              .min(0, "Số lượng không được phép nhỏ hơn 0!")
              .max(9999999999, "Số lượng không được phép lớn hơn 9999999999!");
          },

          export_weight: (val) => {
            if (!val) {
              return validate(val);
            }
            return validate(val)
              .min(0, "Số lượng không được phép nhỏ hơn 0!")
              .max(9999999999, "Số lượng không được phép lớn hơn 9999999999!");
          },
          shifting: (val) => {
            if (!val) {
              return validate(val);
            }
            return validate(val)
              .min(0, "Số lượng không được phép nhỏ hơn 0!")
              .max(9999999999, "Số lượng không được phép lớn hơn 9999999999!");
          },
        });
    },
    checkValidate() {
      const errors = this.validateData(this.forecastVessels)?.error;
      for (let i = 0; i < errors.length; i++) {
        for (const key in errors[i]) {
          if (errors[i][key]) {
            return {
              message: errors[i][key],
              isValid: false,
            };
          }
        }
      }
      return {
        isValid: true,
      };
    },
    changeVesselType2Input() {
      if (this.arrayOfvesselType2.length > 0) {
        this.forecastVessels = this.sotedForecastVessels.filter((element) => {
          if (this.arrayOfvesselType2.includes(element.vesselType2)) {
            return true;
          }
          return false;
        });
      } else {
        this.forecastVessels = this.sotedForecastVessels;
      }
    },
    changeCargosInput() {
      if (this.arayOfCargos.length > 0) {
        let results = [];
        this.forecastVessels = this.sotedCargos.forEach((element) => {
          if (element.cargoType) {
            if (element.cargoType.includes(",")) {
              let tempArr = element.cargoType.split(",");
              tempArr.forEach((el) => {
                if (this.arayOfCargos.includes(el)) {
                  results.push(element);
                }
              });
            } else {
              if (this.arayOfCargos.includes(element.cargoType)) {
                results.push(element);
              }
            }
          }
        });
        // remove duplicates
        this.forecastVessels = results.reduce((unique, o) => {
          if (!unique.some((obj) => obj.id === o.id)) {
            unique.push(o);
          }
          return unique;
        }, []);
      } else {
        this.forecastVessels = this.sotedCargos;
      }
    },
    resetGrid() {
      this.getList();
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
.ve-table
  .ve-table-container
  table.ve-table-content
  tbody.ve-table-body
  tr.ve-table-body-tr
  td.changed-row {
    background-color: $yellow-y1;
  }
.ve-table
  .ve-table-container
  table.ve-table-content
  tbody.ve-table-body
  tr.ve-table-body-tr
  td.blue-row {
    background-color: $green-g1;
  }
.grid-cell-button {
  align-items: center;

  .container-edit-icon .svg-icon {
    width: 20px;
    height: 20px;
  }
}
</style>
