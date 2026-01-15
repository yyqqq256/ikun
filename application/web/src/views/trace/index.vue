<template>
  <div class="trace-container">
    <div class="trace-header">
      <div class="search-section">
        <el-input
          v-model="input"
          placeholder="请输入溯源码查询"
          style="width: 300px; margin-right: 15px;"
        />
        <el-button type="primary" plain @click="FruitInfo">查询</el-button>
        <el-button
          type="success"
          plain
          icon="el-icon-camera"
          @click="showQRScanner = true"
        >
          二维码扫描
        </el-button>
        <el-button
          type="info"
          plain
          icon="el-icon-map-location"
          @click="showSupplyChainMap = true"
        >
          供应链地图
        </el-button>
      </div>
      <el-button type="success" plain @click="AllFruitInfo">获取所有农产品信息</el-button>
    </div>

    <!-- QR扫描对话框 -->
    <el-dialog
      title="二维码扫描溯源"
      :visible.sync="showQRScanner"
      width="600px"
      :before-close="handleQRScannerClose"
    >
      <QRScanner
        @scan-success="handleQRScanSuccess"
        @close="showQRScanner = false"
      />
    </el-dialog>

    <!-- 供应链地图对话框 -->
    <el-dialog
      title="供应链可视化地图"
      :visible.sync="showSupplyChainMap"
      width="900px"
      top="5vh"
    >
      <SupplyChainMap :trace-data="tracedata[0] || {}" />
    </el-dialog>

    <!-- 产品二维码显示对话框 -->
    <ProductQRDisplay
      :visible.sync="showProductQR"
      :traceability-code="currentProduct ? currentProduct.traceability_code : ''"
      :product-name="currentProduct ? currentProduct.farmer_input.fa_fruitName : ''"
      :product-info="currentProduct ? currentProduct.farmer_input : {}"
    />

    <el-table :data="tracedata" style="width: 100%">
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline class="demo-table-expand">
            <div><span class="trace-text" style="color: #67C23A;">农产品信息</span></div>
            <el-form-item label="农产品名称：">
              <span>{{ props.row.farmer_input.fa_fruitName }}</span>
            </el-form-item>
            <el-form-item label="产地：">
              <span>{{ props.row.farmer_input.fa_origin }}</span>
            </el-form-item>
            <el-form-item label="采摘时间：">
              <span>{{ props.row.farmer_input.fa_pickingTime }}</span>
            </el-form-item>
            <el-form-item label="区块链交易ID：">
              <span>{{ props.row.farmer_input.fa_txid }}</span>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>

      <el-table-column label="溯源码" prop="traceability_code">
        <template slot-scope="scope">
          <div class="trace-code-cell">
            <span>{{ scope.row.traceability_code }}</span>
            <el-button
              type="text"
              size="mini"
              class="qr-btn"
              @click="openProductQR(scope.row)"
            >
              <svg-icon icon-class="qrcode" style="margin-right: 2px;" />
              二维码
            </el-button>
          </div>
        </template>
      </el-table-column>

      <el-table-column label="农产品名称" prop="farmer_input.fa_fruitName" />
      <el-table-column label="农产品采摘时间" prop="farmer_input.fa_pickingTime" />
    </el-table>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import {
  getFruitInfo,
  getFruitList,
  getAllFruitInfo
} from '@/api/trace'
import QRScanner from '@/components/QRScanner'
import SupplyChainMap from '@/components/SupplyChainMap'
import ProductQRDisplay from '@/components/ProductQRDisplay'

export default {
  name: 'Trace',
  components: {
    QRScanner,
    SupplyChainMap,
    ProductQRDisplay
  },
  data() {
    return {
      tracedata: [],
      input: '',
      baseApi: process.env.VUE_APP_BASE_API,
      showQRScanner: false,
      showSupplyChainMap: false,
      showProductQR: false,
      currentProduct: null
    }
  },
  computed: {
    ...mapGetters(['name', 'userType'])
  },
  created() {
    const code = this.$route.params.traceability_code
    if (code) {
      this.input = code
      this.FruitInfo()
    } else {
      getFruitList().then(res => {
        this.tracedata = JSON.parse(res.data)
      })
    }
  },
  methods: {
    AllFruitInfo() {
      getAllFruitInfo().then(res => {
        this.tracedata = JSON.parse(res.data)
      })
    },

    FruitInfo() {
      const formData = new FormData()
      formData.append('traceability_code', this.input)
      getFruitInfo(formData).then(res => {
        if (res.code === 200) {
          this.tracedata = [JSON.parse(res.data)]
        } else {
          this.$message.error(res.message)
        }
      })
    },

    handleQRScanSuccess(qrCode) {
      this.input = qrCode
      this.showQRScanner = false
      this.FruitInfo()
      this.$message.success(`二维码扫描成功: ${qrCode}`)
    },

    handleQRScannerClose() {
      this.showQRScanner = false
    },

    openProductQR(product) {
      this.currentProduct = product
      this.showProductQR = true
    }
  }
}
</script>

<style lang="scss" scoped>
.trace-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;

  .search-section {
    display: flex;
    align-items: center;
  }
}

.trace-container {
  margin: 30px;
}

.trace-text {
  font-size: 30px;
  line-height: 46px;
}

.trace-code-cell {
  display: flex;
  align-items: center;
  justify-content: space-between;

  .qr-btn {
    margin-left: 10px;
    padding: 2px 8px;
    font-size: 12px;
  }
}

.demo-table-expand {
  font-size: 0;

  label {
    width: 90px;
    color: #99a9bf;
  }

  .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
    display: inline-block;
    vertical-align: top;
  }
}
</style>
