<template>
  <el-dialog
    :visible.sync="updateTimeForm.open"
    v-if="updateTimeForm.open"
    v-loading="loading"
    append-to-body
    custom-class="dialog__update-time"
    :destroy-on-close="true"
  >
    <div slot="title" class="custom-title">
      <svg-icon icon-class="dialog-edit" />
      <span class="title">{{ updateTimeForm.title }}</span>
    </div>
    <div class="content-updateTime">
      <el-form
        class="dialog-content"
        ref="form"
        :model="form"
        :rules="rules"
        label-width="100px"
      >
        <div class="dialog-content-detail">
          <div class="detail-title detail-title-group">
            <span>Tên tàu: {{ updateTimeForm.vessel.vesselName }}</span>
            <span style="padding: 0 5px">-</span
            ><span>Bến: {{ updateTimeForm.vessel.berthNo }}</span>
          </div>
          <div class="detail-content form__berth form__spacing-item">
            <el-row v-hasPermi="['bm:berth-plan:plan-time']">
              <el-col :span="24">
                <el-form-item label="ETA" prop="eta">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.eta"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:plan-time']">
              <el-col :span="24">
                <el-form-item label="ETB" prop="etb">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.etb"
                    @change="etb"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:plan-time']">
              <el-col :span="24">
                <el-form-item label="ETW" prop="etw">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.etw"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:plan-time']">
              <el-col :span="24">
                <el-form-item label="ETC" prop="etc">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.etc"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:plan-time']">
              <el-col :span="24">
                <el-form-item label="ETD" prop="etd">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.etd"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-if="disabledATA ? disabledATA : isForecast">
              <el-col :span="24">
                <el-form-item label="ATA" prop="ata">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.ata"
                    :disabled="disabledATA"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:actual-time']">
              <el-col :span="24">
                <el-form-item label="ATB" prop="atb">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.atb"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:actual-time']">
              <el-col :span="24">
                <el-form-item label="ATW" prop="atw">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.atw"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:actual-time']">
              <el-col :span="24">
                <el-form-item label="ATC" prop="atc">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.atc"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-hasPermi="['bm:berth-plan:actual-time']">
              <el-col :span="24">
                <el-form-item label="ATD" prop="atd">
                  <el-date-picker
                    style="width: 100%"
                    type="datetime"
                    format="dd/MM/yyyy HH:mm"
                    value-format="yyyy-MM-dd HH:mm"
                    v-model="form.atd"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-col>
            </el-row>
          </div>
        </div>
      </el-form>
    </div>
    <div slot="footer" class="custom-footer">
      <minimal-button
        :width="'110px'"
        :title="'Hủy bỏ'"
        :iconPosition="'icon-left'"
        @click="closeForm"
      />
      <primary-button
        :width="'90px'"
        :title="'Lưu'"
        :iconPosition="'icon-left'"
        @click="saveForm"
      />
    </div>
  </el-dialog>
</template>
<script>
import MinimalButton from "@/components/CustomButtons/MinimalButton";
import PrimaryButton from "@/components/CustomButtons/PrimaryButton";
import { listBerthNotPage } from "../../../api/bm/berthList";
import { updateVesselPlan, getInfoByBerthPlanId } from "@/api/bm/vesselPlan";
import moment from "moment";
import { confirmDialogRender } from "@/utils/render";
import { checkPermi } from "@/utils/permission";

export default {
  props: ["updateTimeForm", "isForecast"],
  components: {
    MinimalButton,
    PrimaryButton,
  },
  data() {
    var validateEtb = (rule, value, callback) => {
      if (this.etb && !value) {
        return callback(new Error("Ngày ETB không được phép trống!"));
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.eta &&
          moment(this.form.eta).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(new Error("Thời gian phải lớn hơn/bằng ETA!"));
        } else {
          callback();
        }
      }
    };
    var validateEtw = (rule, value, callback) => {
      if (this.etw && !value) {
        return callback(new Error("Ngày ETW không được phép trống!"));
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.etb &&
          moment(this.form.etb).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(new Error("Thời gian phải lớn hơn/bằng ETA,ETB!"));
        } else {
          callback();
        }
      }
    };
    var validateEtc = (rule, value, callback) => {
      if (this.etc && !value) {
        return callback(new Error("Ngày ETC không được phép trống!"));
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.etw &&
          moment(this.form.etw).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(
            new Error("Thời gian phải lớn hơn/bằng ETA,ETB,ETW!")
          );
        } else {
          callback();
        }
      }
    };
    var validateEtd = (rule, value, callback) => {
      if (this.etd && !value) {
        return callback(new Error("Ngày ETD không được phép trống!"));
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.etc &&
          moment(this.form.etc).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(
            new Error("Thời gian phải lớn hơn/bằng ETA,ETB,ETW,ETC!")
          );
        } else {
          callback();
        }
      }
    };
    var validateAtb = (rule, value, callback) => {
      if (!value) {
        return callback();
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.ata &&
          moment(this.form.ata).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(new Error("Thời gian phải lớn hơn/bằng ATA!"));
        } else {
          callback();
        }
      }
    };
    var validateAtw = (rule, value, callback) => {
      if (!value) {
        return callback();
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.atb &&
          moment(this.form.atb).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(new Error("Thời gian phải lớn hơn/bằng ATA,ATB!"));
        } else {
          callback();
        }
      }
    };
    var validateAtc = (rule, value, callback) => {
      if (!value) {
        return callback();
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.atw &&
          moment(this.form.atw).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(
            new Error("Thời gian phải lớn hơn/bằng ATA,ATB,ATW!")
          );
        } else {
          callback();
        }
      }
    };
    var validateAtd = (rule, value, callback) => {
      if (!value) {
        return callback();
      } else {
        let date = moment(value).format("YYYY-MM-DD HH:mm");
        if (
          this.form.atc &&
          moment(this.form.atc).format("YYYY-MM-DD HH:mm") > date
        ) {
          return callback(
            new Error("Thời gian phải lớn hơn/bằng ATA,ATB,ATW,ATC!")
          );
        } else {
          callback();
        }
      }
    };
    return {
      etb: null,
      etw: null,
      etc: null,
      etd: null,
      totalNormTime: 0,
      isShow: true,
      berthList: [],
      form: {},
      loading: false,
      // Form validation
      rules: {
        eta: [
          {
            required: true,
            message: "Ngày ETA không được phép trống!",
            trigger: "blur",
          },
        ],
        etb: [
          {
            validator: validateEtb,
            trigger: "blur",
          },
        ],
        etw: [
          {
            validator: validateEtw,
            trigger: "blur",
          },
        ],
        etc: [
          {
            validator: validateEtc,
            trigger: "blur",
          },
        ],
        etd: [
          {
            validator: validateEtd,
            trigger: "blur",
          },
        ],
        atb: [
          {
            validator: validateAtb,
            trigger: "blur",
          },
        ],
        atw: [
          {
            validator: validateAtw,
            trigger: "blur",
          },
        ],
        atc: [
          {
            validator: validateAtc,
            trigger: "blur",
          },
        ],
        atd: [
          {
            validator: validateAtd,
            trigger: "blur",
          },
        ],
      },
    };
  },
  watch: {
    async updateTimeForm(newValue) {
      if (newValue.open) {
        if (newValue.vessel) {
          this.totalNormTime = 0;
          this.getTotalNormTime(newValue.vessel.id);
          this.form = newValue.vessel;
          this.etb = newValue.vessel.etb;
          this.etw = newValue.vessel.etw;
          this.etc = newValue.vessel.etc;
          this.etd = newValue.vessel.etd;
        }
      }
    },
    etb() {
      console.log(this.form.etb);
      const totalHours = parseFloat(this.totalNormTime);
      if (isNaN(totalHours)) return;

      const etbDate = new Date(this.etb);
      const etdDate = new Date(etbDate.getTime() + totalHours * 3600 * 1000);

      console.log(etdDate);
      this.etd = etdDate.toISOString().substring(0, 16);
      this.form.etd = this.etd;
    },
  },
  computed: {
    disabledATA() {
      return checkPermi(["bm:berth-plan:actual-time"]);
    },
  },
  methods: {
    closeForm() {
      this.$emit("close-form");
    },
    // generateEtd(etb) {
    //   if (etb) {
    //     const etbDate = new Date(this.etb.replace(' ', 'T'));

    //     // Tính toán số giờ tương ứng với totalNormTime
    //     const totalHours = this.totalNormTime;

    //     // Thêm số giờ vào ngày ETA
    //     etbDate.setHours(etbDate.getHours() + totalHours);

    //     // Chuyển đổi ngày ETD thành chuỗi định dạng dd/mm/yyyy
    //     const etdDate = etbDate.toISOString().slice(0, 16).replace('T', ' ');
    //     console.log(etdDate);
    //     this.etd = etdDate;
    //   } else {
    //     this.etd = null;
    //   }
    // },
    getTotalNormTime(berthPlanId) {
      getInfoByBerthPlanId(berthPlanId).then((response) => {
        if (response.data) {
          response.data.forEach((obj) => {
            this.totalNormTime += obj.normTime;
          });
        }
      });
    },
    getBerths() {
      listBerthNotPage().then((res) => {
        if (res.code == 200) {
          this.berthList = res.data;
        }
      });
    },
    getData() {},
    saveForm() {
      this.$refs["form"].validate((valid) => {
        if (valid) {
          let data = this.form;
          data.listBMBerthPlanOther = [];
          data.listBMBerthShiftings = [];
          if (data.etb) {
            const etb = moment(data.etb, "YYYY-MM-DD HH:mm");
            data.yardClose = etb
              .subtract(3, "hours")
              .format("YYYY-MM-DD HH:mm");
          }
          // to do gọi api để lưu
          updateVesselPlan(data)
            .then((response) => {
              this.responseMessage(
                "Cập nhật thời gian thành công!",
                this.$MESSAGE_TYPE.SUCCESS
              );
              this.updateTimeForm.open = false;
              this.$emit("reset-grid");
            })
            .catch((error) => {
              this.responseMessage(error.message, this.$MESSAGE_TYPE.ERROR);
              this.updateTimeForm.open = true;
            });
        }
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
<style lang="scss">
.dialog__update-time {
  .el-form {
    padding: 0;
  }
  .el-form-item {
    margin-bottom: 0;
  }
  .detail-title-group {
    font-weight: 500;
    font-size: 18px;
    line-height: 24px;
    color: #7c889e;
  }

  .content-updateTime {
    display: flex;
    flex-direction: column;
    gap: 21px;
  }
}
</style>
