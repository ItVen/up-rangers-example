<template>
  <div id="page-demo" class="unipass-page">
    <i class="background-logo iconfont icon-logo"></i>
    <div class="head">UniPass Demo</div>
    <div v-if="username">
      <div>
        <br />
        <h3>{{ username }}</h3>
        <br />
      </div>
      <el-button class="transfer" type="primary" @click="logout"
        >logout</el-button
      >
    </div>
    <div v-else>
      <br />
      <el-radio v-model="toTheme" label="dark">Dark</el-radio>
      <el-radio v-model="toTheme" label="light">Light</el-radio>
      <el-button type="primary" class="transfer login" @click="connect">
        login
      </el-button>
    </div>
    <el-tabs
      v-show="username"
      v-model="activeTab"
      class="body"
      type="border-card"
    >
      <el-tab-pane label="RPG Transaction" name="first">
        <el-form
          ref="form"
          class="body-input"
          label-position="top"
          :model="form"
          @submit.native.prevent
        >
          <el-form-item label="Your Address">
            <template #label>
              <span>Your Address</span>
              <i
                v-show="myAddress"
                class="iconfont icon-copy sea-background"
                @click="bindCopy"
              ></i>
            </template>
            <el-input
              v-model="myAddress"
              disabled
              readonly
              type="textarea"
              resize="none"
              :autosize="{ minRows: 1 }"
            />
          </el-form-item>
          <el-form-item label="Your Balance">
            <!-- <el-input v-model="myBalanceFormat" disabled readonly /> -->
            <div class="balance-box">
              <up-balance name="BNB" />
              <div class="right">0.02</div>
            </div>
            <div class="balance-box">
              <up-balance name="ETH" />
              <div class="right">0.02</div>
            </div>
            <div class="balance-box">
              <up-balance name="USDT" />
              <div class="right">0.02</div>
            </div>
            <div class="balance-box">
              <up-balance name="USDC" />
              <div class="right">0.02</div>
            </div>
          </el-form-item>
          <el-form-item label="To">
            <el-input v-model="toAddress" placeholder="Address" clearable />
          </el-form-item>
          <el-form-item label="Token" prop="token" class="token">
            <el-select
              v-model="tokenSelect"
              placeholder="请选择"
              popper-class="token"
            >
              <el-option
                v-for="item in tokens"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
                <img :src="item.icon" :alt="item.value" />
                <span>{{ item.label }}</span>
              </el-option>
            </el-select>
            <el-input
              v-model="toAmount"
              type="number"
              placeholder="Amount"
            ></el-input>
          </el-form-item>
          <el-form-item label="Fee">
            <el-radio
              v-for="(balance, i) in balanceList"
              :key="i"
              v-model="toToken"
              :label="balance.name"
              class="token-radio"
            >
              <div class="balance-box">
                <up-balance :name="balance.name" />
                <div class="right">
                  <span>{{ balance.fee }}</span>
                  <div class="dot-box">
                    <div v-show="toToken === balance.name" class="dot"></div>
                  </div>
                </div>
              </div>
            </el-radio>
          </el-form-item>
          <!-- <el-form-item label="Amount:" prop="address">
            <el-input v-model="toAmount" clearable />
          </el-form-item>
          <el-form-item label="Fee:" prop="fee">
            <el-input v-model="toFeeAmount" clearable />
          </el-form-item>
          <el-form-item label="Description:" prop="description">
            <el-input v-model="toDescription" clearable />
          </el-form-item> -->
        </el-form>
        <br />
        <div>
          <el-button type="primary" class="transfer" @click="sendRPG">
            sendRPG
          </el-button>
          <br />
          <br />
          <el-button class="transfer" @click="sendToken"> sendToken </el-button>
          <br />
          <br />
          <el-button type="primary" class="transfer" @click="executeCall">
            executeCall
          </el-button>
        </div>
        <div>{{ txHash }}</div>
      </el-tab-pane>
      <el-tab-pane label="Sign Message" name="second">
        <div>
          <br />
          <h3 class="input">Message:</h3>
          <el-input
            v-model="message"
            type="textarea"
            :autosize="{ minRows: 8, maxRows: 10 }"
            resize="none"
          ></el-input>
          <br />
          <div class="message">
            <el-button type="primary" class="message-button" @click="authorize"
              >authorize</el-button
            >
            <el-button type="primary" class="message-button" @click="verifySig"
              >verify</el-button
            >
          </div>
          <br />
          <div v-if="sig">
            <h3 class="input">Signature:</h3>
            <el-input
              v-model="sig"
              type="textarea"
              :autosize="{ minRows: 8, maxRows: 10 }"
              resize="none"
            ></el-input>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { UPAuthMessage, UPAuthResponse } from 'up-core-test'
import { ChainID, UPRangers } from 'up-rangers'

const DAI_ADDRESS = '0x25c58Aa062Efb4f069bD013De3e3C5797fb40651'

export default Vue.extend({
  data() {
    return {
      username: '',
      message: 'TO BE SIGNED MESSAGE abc',
      sig: '',
      activeTab: 'first',
      myAddress: '',
      myBalance: '0.00',
      toAddress: '',
      // toAddress: '0x8291507Afda0BBA820efB6DFA339f09C9465215C',
      toAmount: '',
      toFeeAmount: '0.000001',
      toDescription: '描述测试描述测试描述测试',
      toTheme: 'dark',
      toToken: 'BNB',
      txHash: '',
      balanceList: [
        {
          name: 'BNB',
          fee: 0.01,
        },
        {
          name: 'ETH',
          fee: 0.0001,
        },
        {
          name: 'USDT',
          fee: 0.1,
        },
        {
          name: 'USDC',
          fee: 0.1,
        },
      ],
      form: {},
      tokenSelect: 'ETH',
      tokens: [
        {
          value: 'ETH',
          label: 'ETH',
          icon: require('@/assets/img/ETH.svg'),
        },
        {
          value: 'BNB',
          label: 'BNB',
          icon: require('@/assets/img/BNB.svg'),
        },
      ],
      // STEP 1: create UPRangers instance
      upRangers: new UPRangers({
        chainID: ChainID.testnet,
        userInfoContract: process.env.RANGERS_UNIPASS_CONTRACT,
        upCoreConfig: {
          domain: process.env.UNIPASS_URL,
          // domain: 'localhost:3000',
          // protocol: 'http',
        },
      }),
    }
  },
  computed: {
    myBalanceFormat(): string {
      return this.myBalance + ' RPG'
    },
  },
  methods: {
    bindCopy() {
      this.$clipboard(this.myAddress)
      this.$message.success('copy succeeded')
    },
    async connect() {
      console.log('connect clicked')
      try {
        // STEP 2: connect unipass
        const account = await this.upRangers.getUPCore().connect({
          email: true,
          evmKeys: true,
          chain: '1',
          theme: this.toTheme as any,
        })
        this.username = account.username
        console.log('account', account)

        // STEP 3: init unipass with username and email
        await this.upRangers.initUniPass(this.username, account.email!)

        this.myAddress = this.upRangers.getAddress()
        await this.refreshBalance()
      } catch (err) {
        this.$message.error(err as string)
        console.log('connect err', err)
      }
    },
    async refreshBalance() {
      this.myBalance = await this.upRangers
        .getWeb3()
        .eth.getBalance(this.myAddress)
    },
    logout() {
      console.log('connect clicked')
      this.upRangers.getUPCore().disconnect()
      this.username = ''
    },
    async authorize() {
      console.log('authorize clicked')
      this.sig = ''
      console.log({
        username: this.username,
        message: this.message,
      })
      try {
        // SIGN Message with UniPass
        const resp = await this.upRangers
          .getUPCore()
          .authorize(
            new UPAuthMessage('PLAIN_MSG', this.username, this.message),
          )
        console.log('resp', resp)
        this.sig = JSON.stringify(resp)
      } catch (err) {
        this.$message.error(err as string)
        console.log('auth err', err)
      }
    },

    async verifySig() {
      try {
        // VERIFY user signed message and sig
        const ret = await this.upRangers.verifyUserSig(
          '0x' + Buffer.from(this.message, 'utf-8').toString('hex'),
          JSON.parse(this.sig) as UPAuthResponse,
        )
        if (ret === true) {
          this.$message.success('verify signature success')
        } else {
          this.$message.error('verify signature failed')
        }
      } catch (err) {
        this.$message.error(err as string)
        console.log('auth err', err)
      }
    },
    async sendRPG() {
      if (Number(this.myBalance) < Number(this.toAmount)) {
        this.$message.error('balance is not enough')
        return
      }
      try {
        this.upRangers.getUPCore().initPop()

        // SEND RPG
        this.txHash = await this.upRangers.transferNativeToken(
          this.toAddress,
          this.upRangers.getWeb3().utils.toWei(this.toAmount),
          {
            feeToken: {
              address: '0x0000000000000000000000000000000000000000',
              symbol: 'RPG',
              decimals: 18,
            },
            feeAmount: this.upRangers.getWeb3().utils.toWei(this.toFeeAmount),
            description: this.toDescription,
          },
        )
        console.log('send RPG success', this.txHash)
        this.$message.success(`send RPG success, tx hash = ${this.txHash}`)

        await this.refreshBalance()
      } catch (err) {
        this.$message.error(err as string)
        console.log('err', err)
      }
    },
    async sendToken() {
      try {
        this.upRangers.getUPCore().initPop()

        // SEND DAI(ERC20) token
        this.txHash = await this.upRangers.transferToken(
          {
            address: DAI_ADDRESS,
            symbol: 'DAI',
            decimals: 18,
          },
          this.toAddress,
          this.upRangers.getWeb3().utils.toWei(this.toAmount),
          {
            feeToken: {
              address: '0x0000000000000000000000000000000000000000',
              symbol: 'RPG',
              decimals: 18,
            },
            feeAmount: this.upRangers.getWeb3().utils.toWei(this.toFeeAmount),
            description: this.toDescription,
          },
        )

        console.log('send Token success', this.txHash)
        this.$message.success(`send token success, tx hash = ${this.txHash}`)
      } catch (err) {
        this.$message.error(err as string)
        console.log('err', err)
      }
    },
    async executeCall() {
      try {
        this.upRangers.getUPCore().initPop()

        // CALL CONTRACT to execute its method
        this.txHash = await this.upRangers.executeCall(
          DAI_ADDRESS,
          '0x00',
          this.upRangers.getWeb3().eth.abi.encodeFunctionCall(
            {
              name: 'transfer',
              type: 'function',
              inputs: [
                {
                  name: 'dst',
                  type: 'address',
                },
                {
                  name: 'wad',
                  type: 'uint256',
                },
              ],
            },
            [
              this.toAddress,
              this.upRangers.getWeb3().utils.toWei(this.toAmount),
            ],
          ),
          {
            feeToken: {
              address: '0x0000000000000000000000000000000000000000',
              symbol: 'RPG',
              decimals: 18,
            },
            feeAmount: this.upRangers.getWeb3().utils.toWei(this.toFeeAmount),
            description: this.toDescription,
          },
        )
        console.log('execute call success', this.txHash)
        this.$message.success(`execute call success, tx hash = ${this.txHash}`)
      } catch (err) {
        this.$message.error(err as string)
        console.log('err', err)
      }
    },
  },
})
</script>

<style lang="scss">
#page-demo {
  max-width: 480px;
  margin: 0 auto;
  overflow: hidden;
  position: relative;
  background: #f5f5ff;

  > * {
    z-index: 1;
  }

  > .background-logo {
    font-size: 237px;
    position: absolute;
    top: 16px;
    right: -40px;
    color: #5575ff;
    opacity: 0.14;
    z-index: 0;
  }

  .head {
    text-align: left;
    font-family: Helvetica;
    font-style: normal;
    font-weight: bold;
    font-size: 20px;
    line-height: 20px;
    color: black;
  }

  .transfer {
    width: 100%;
    font-size: 20px;
  }

  .login {
    margin-top: 50px;
    font-size: 20px;
  }

  .body {
    border-radius: 24px;
    margin: 30px auto 0px;
    width: 100%;
    background: #ffffff;
    padding: 0px 0 21px;
    overflow: hidden;

    .body-input {
      margin-top: -20px;

      .icon-copy {
        cursor: pointer;
      }
    }

    .input {
      text-align: left;
      margin-bottom: 20px;
    }
  }

  .message {
    display: flex;
    justify-content: space-between;

    .message-button {
      margin-top: 30px;
      width: 48%;
      font-size: 20px;
    }
  }
  .token-radio {
    margin-right: 0;
    .el-radio__input {
      display: none;
    }
    .el-radio__label {
      width: 100%;
      padding: 0;
    }
  }
  .token-radio + .token-radio {
    margin-top: 15px;
  }
  .balance-box {
    padding: 12px;
    width: 100%;
    background: #f5f5f7;
    border-radius: 10px;
    font-size: 16px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    .right {
      display: flex;
      align-items: center;
      .dot-box {
        margin-left: 8px;
        display: flex;
        justify-content: center;
        align-items: center;
        width: 16px;
        height: 16px;
        background: #ffffff;
        border: 1px solid #c1c1c1;
        border-radius: 50%;
        .dot {
          border-radius: 50%;
          width: 8px;
          height: 8px;
          background: #0364ff;
        }
      }
    }
  }
  .balance-box + .balance-box {
    margin-top: 15px;
  }
  .el-form-item.token {
    .el-form-item__content {
      display: flex;
      .el-select {
        margin-right: 10px;
      }
    }
  }
}

.unipass-page {
  padding: 24px;
  padding-top: 29px;
  width: 100%;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  text-align: center;
}
.el-select-dropdown.token {
  .el-select-dropdown__item {
    display: flex;
    align-items: center;
    img {
      margin-right: 8px;
    }
  }
}
</style>
