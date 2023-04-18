<template>
  <div class="dialog-content-detail vessel-info-form">
    <div class="detail-title">Thông tin tàu</div>
    <div class="detail-content form__spacing-item">
      <div class="form-item-list">
        <el-row class="form__group-title">
          <el-col :span="6">Thông tin chi tiết</el-col>
          <el-col :span="6">Thông tin chung</el-col>
          <el-col :span="6">Sức chứa</el-col>
          <el-col :span="6">Kích thước</el-col>
        </el-row>
        <el-row>
          <el-col :span="6">
            <el-form-item
              label-width="120px"
              label="Tên tàu/ Name of ship"
              prop="vesselName"
              class="validate-padding"
            >
              <el-autocomplete
                v-model="form.vesselName"
                :fetch-suggestions="querySearch"
                placeholder="Nhập nội dung"
                style="width: 100%"
                clearable
                @clear="clearForm"
                @select="handleSelectVessel"
                :disabled="isDisabled"
              >
                <template slot-scope="scope">
                  <span :title="scope.item">{{ scope.item }}</span>
                </template>

                <primary-button
                  icon="transfer"
                  :iconPosition="'icon-left'"
                  type="button"
                  slot="append"
                  levelColor="6"
                  @click="getDataFromCatos(form.vesselName)"
                ></primary-button>
              </el-autocomplete>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Hãng tàu/ Shipping line"
              label-width="120px"
              prop="shippingLine"
            >
              <el-input
                v-model="form.shippingLine"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="8"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Dung tích có ích/ Net weight"
              label-width="120px"
              prop="netWeight"
            >
              <el-input
                v-model="form.netWeight"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Chiều dài toàn bộ/ LOA"
              label-width="120px"
              prop="loa"
            >
              <el-input
                v-model="form.loa"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="8"
              >
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="6">
            <el-form-item
              label-width="120px"
              label="Mã tàu/ Ship ID"
              prop="vesselCode"
            >
              <el-input
                type="text"
                v-model="form.vesselCode"
                class="input-temp2"
                placeholder="Nhập"
                :disabled="isDisabled || disabledCatos"
                maxlength="10"
                @input="handleVesselCodeUpperCase($event)"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Quốc tịch/ Flag"
              label-width="120px"
              prop="citizenshipName"
            >
              <el-autocomplete
                class="input-temp"
                v-model="form.citizenshipName"
                :fetch-suggestions="querySearchCitizenship"
                placeholder="Chọn"
                @select="handleCitizenship"
                style="width: 100%"
                clearable
              >
              </el-autocomplete>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Dung tích toàn phần/ Gross weight"
              label-width="120px"
              prop="grossWeight"
            >
              <el-input
                v-model="form.grossWeight"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Chiều dài tính toán/ LBP"
              label-width="120px"
              prop="lbp"
            >
              <el-input
                v-model="form.lbp"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="6">
            <el-form-item
              label="Loại tàu 1/ Shipping type 1"
              label-width="120px"
              prop="vesselType1"
            >
              <el-select
                v-model="form.vesselType1"
                placeholder="Chọn"
                class="input-temp"
              >
                <el-option
                  v-for="item in listVesseTypel"
                  :key="item"
                  :label="item"
                  :value="item"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Hô hiệu/ Call sign"
              label-width="120px"
              prop="callSign"
            >
              <el-input
                class="input-temp"
                v-model="form.callSign"
                placeholder="Nhập"
                type="text"
                maxlength="255"
                @input="handleCallSignUpperCase($event)"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Trọng tải toàn phần/ Dead weight"
              label-width="120px"
              prop="deadWeight"
            >
              <el-input
                v-model="form.deadWeight"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Chiều rộng/ Breadth"
              label-width="120px"
              prop="width"
            >
              <el-input
                v-model="form.width"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="6">
            <el-form-item
              label="Loại tàu 2/ Shipping type 2"
              label-width="120px"
              prop="vesselType2"
            >
              <el-select
                v-model="form.vesselType2"
                placeholder="Nhập"
                class="input-temp"
                :disabled="form.vesselType2 == 'Container' || disabledCatos"
              >
                <el-option
                  v-for="item in listVesseType2"
                  :key="item"
                  :label="item"
                  :value="item"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Số đăng kiểm/ Inmarsat"
              label-width="120px"
              prop="inmarsat"
            >
              <el-input
                class="input-temp"
                v-model="form.inmarsat"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Số hầm/ Hatch quantity"
              label-width="120px"
              prop="hatchQuantity"
            >
              <el-input
                class="input-temp"
                v-model="form.hatchQuantity"
                placeholder="Nhập"
                type="text"
                maxlength="3"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Chiều cao/ Height"
              label-width="120px"
              prop="depth"
            >
              <el-input
                class="input-temp"
                v-model="form.depth"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="6">
            <el-form-item
              label="Loại tàu 3/ Shipping type 3"
              label-width="120px"
              prop="vesselType3"
            >
              <el-select
                v-model="form.vesselType3"
                placeholder="Chọn"
                class="input-temp"
              >
                <el-option
                  v-for="item in listVesseType3"
                  :key="item"
                  :label="item"
                  :value="item"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Lloyd/ Số điện thoại vệ tinh"
              label-width="120px"
              prop="lloyd"
            >
              <el-input
                type="text"
                v-model="form.lloyd"
                class="input-temp"
                placeholder="Nhập"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Số cẩu tàu/ Crane quantity"
              label-width="120px"
              prop="craneQuantity"
            >
              <el-input
                v-model="form.craneQuantity"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="3"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Mớn nước tối đa/ Max Draft"
              label-width="120px"
              prop="maxDraft"
            >
              <el-input
                v-model="form.maxDraft"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="6">
            <el-form-item
              label="Ghi chú/ Remark"
              label-width="120px"
              prop="remark"
            >
              <el-input
                v-model="form.remark"
                type="textarea"
                class="input-temp"
                placeholder="Nhập"
                maxlength="255"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Thuyền trưởng/ Ship Owner"
              label-width="120px"
              prop="shipOwner"
            >
              <el-input
                v-model="form.shipOwner"
                class="input-temp"
                placeholder="Nhập"
                type="text"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
              label="Tải trọng cẩu tàu/ Crane capacity"
              label-width="120px"
              prop="craneCapacity"
            >
              <el-input
                v-model="form.craneCapacity"
                class="input-temp2"
                placeholder="Nhập"
                type="text"
                maxlength="16"
              >
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6"> </el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>
<script>
import {
  getFastPartnerList,
  getVesselListFromCatos,
  listCitizenship,
  getVesselNameList,
} from "@/api/masterDataApi/masterDataApi.js";
import PrimaryButton from "@/components/CustomButtons/PrimaryButton";

export default {
  props: ["formModel", "rules", "isDisabled"],
  components: { PrimaryButton },
  data() {
    return {
      listPartnerRender: [],
      // list citizenship
      listCitizenship: [],
      listVesseTypel: ["Domestic", "Foreign"],
      listVesseType2: [
        "Multi-purpose",
        "Bulk carrier",
        "RO/RO vessel",
        "Barge",
        "Passenger ship",
      ],
      listVesseType3: ["Just in time", "Weekly", "Liner", "Tramper"],
      // search input
      queryParamSubmit: {
        params: {
          keySearch: null,
        },
      },
      disabledCatos: false,
      vesselName: [],
    };
  },
  created: async function () {
    this.vesselId = this.$route.params.id;
  },

  computed: {
    form: {
      get() {
        return this.formModel;
      },
      set(value) {
        this.$emit("input", value);
      },
    },
  },
  mounted() {
    this.getListVesselName();
  },
  methods: {
    clearForm() {
      this.form = {
        vesselName: "",
        vesselCode: "",
      };
    },
    handleVesselNameUpperCase(e) {
      this.form.vesselName = e.toUpperCase();
    },
    handleCallSignUpperCase(e) {
      this.form.callSign = e.toUpperCase();
    },
    handleVesselCodeUpperCase(e) {
      this.form.vesselCode = e.toUpperCase();
    },

    handleCitizenship(item) {
      this.form.flag = item.link;
    },
    getDataFromCatos(vesselName) {
      if (vesselName != undefined) {
        this.getData(this.form.vesselName);
      } else {
        this.$emit(
          "response-message",
          "Vui lòng nhập: Tên tàu/ Name of ship !",
          this.$MESSAGE_TYPE.WARNING
        );
      }
    },
    getData(keySearch) {
      getVesselListFromCatos(keySearch).then((response) => {
        if (response) {
          const form = {
            vesselCode: response.vslCd,
            vesselName: response.vslNm,
            vesselType2: "Container",
            remark: response.remark,
            // shippingLine: response.shippingLine,
            // flag: response.flag,
            callSign: response.callSign,
            inmarsat: response.inmarsat,
            lloyd: response.lloydCode,
            shipOwner: response.owner,
            netWeight: response.netWeight,
            grossWeight: response.grossWeight,
            deadWeight: response.dwgt,
            loadedDisplacement: response.disp,
            hatchQuantity: response.hatchQuantity,
            craneQuantity: response.craneQuantity,
            loa: response.loa,
            lbp: response.lbp,
            width: response.width,
            airDraft: response.airDraft,
            depth: response.depth,
            craneCapacity: response.disp,
            maxDraf: response.maxDraft,
            systemType: "CATOS",
          };
          this.disabledCatos = true;
          this.setForm(form);
        } else {
          this.$emit(
            "response-message",
            "Tên tàu/ Name of ship không có trong hệ thống !",
            this.$MESSAGE_TYPE.WARNING
          );
          const formReset = {};
          this.setForm(formReset);
        }
      });
    },
    querySearch(queryString, cb) {
      this.queryParamSubmit.params.keySearch = queryString;
      this.listPartnerRender = [];
      this.getPartnerList(this.queryParamSubmit).then((res) => {
        cb(res);
        this.$emit("partner-list", res);
      });
    },
    getPartnerList(data) {
      return getFastPartnerList(data).then((response) => {
        if (response.code == 200) {
          this.listPartnerRender = response.data.map((el) => {
            return {
              value: el.partnerName,
              link: el.partnerCode,
            };
          });
        }
        return this.listPartnerRender;
      });
    },
    handleSelectShippingLine(item) {},
    setForm(form) {
      this.$emit("set-form", form);
    },
    querySearchCitizenship(queryString, cb) {
      this.queryParamSubmit.params.keySearch = queryString;
      this.listCitizenship = [];
      this.getCitizenshipList(this.queryParamSubmit).then((res) => cb(res));
    },
    getCitizenshipList(data) {
      return listCitizenship(data).then((response) => {
        if (response.length > 0) {
          this.listCitizenship = response.map((el) => {
            return {
              value: el.citizenshipName,
              link: el.citizenship,
            };
          });
        }
        return this.listCitizenship;
      });
    },
    querySearch(queryString, cb) {
      let result = [];
      let query = "";
      if (queryString) {
        query = queryString;
      }
      this.getListVesselName();
      this.vesselName.forEach((item) => {
        if (item.includes(query)) {
          result.push(item);
        }
      });
      cb(result);
    },
    getListVesselName() {
      getVesselNameList().then((res) => {
        this.vesselName = res.data;
      });
    },
    handleSelectVessel(item) {
      this.form.vesselName = item;
    },
  },
};
</script>
<style lang="scss">
@import "@/assets/styles/base-variables.scss";

.vessel-info-form {
  border-radius: 0px 0px 8px 8px;
  .el-input-group__append {
    background: transparent;
    border: none;
    padding: 0 0 0 8px;
  }
  .form__spacing-item {
    overflow: auto;

    &::-webkit-scrollbar {
      height: 6px;
      width: 6px;
      border-radius: 100px;
      border: none;
    }
    &::-webkit-scrollbar-track {
      background: rgba(0, 0, 0, 0);
      display: none;
    }
    &::-webkit-scrollbar-thumb {
      background: $neutral-n5;
      border-radius: 100px;
      border: none;
      &:hover {
        background: $neutral-n6;
      }
      &:active {
        background: $neutral-n7;
      }
    }
    &::-webkit-scrollbar-button {
      display: none;
    }
  }

  .form-item-list {
    min-width: 1700px;
  }

  .form__group-title {
    font-weight: 600;
    font-size: 16px;
    line-height: 24px;
    color: $neutral-n8;
    margin-bottom: 24px;
  }
  .detail-title {
    background: $neutral-n0;
    border-bottom: 1px solid $neutral-n2;
    border-radius: 8px 8px 0px 0px;
  }
}
</style>
