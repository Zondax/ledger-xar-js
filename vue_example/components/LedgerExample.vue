<template>
  <div class="ledger">
    <input id="webusb" v-model="transportChoice" type="radio" value="WebUSB" />
    <label for="webusb">WebUSB</label>
    <input id="u2f" v-model="transportChoice" type="radio" value="U2F" />
    <label for="u2f">U2F</label>
    <br />
    <!--
        Commands
    -->
    <button @click="getVersion">
      Get Version
    </button>

    <button @click="getPublicKey">
      Get pubkey only
    </button>

    <button @click="getAddress">
      Get Address and Pubkey
    </button>

    <button @click="showAddress">
      Show Address and Pubkey
    </button>

    <button @click="signExampleTx">
      Sign Example TX
    </button>
    <!--
        Commands
    -->
    <ul id="ledger-status">
      <li v-for="item in ledgerStatus" :key="item.index">
        {{ item.msg }}
      </li>
    </ul>
  </div>
</template>

<script>
// eslint-disable-next-line import/no-extraneous-dependencies
import TransportWebUSB from "@ledgerhq/hw-transport-webusb";
// eslint-disable-next-line import/no-extraneous-dependencies
import TransportU2F from "@ledgerhq/hw-transport-u2f";
import XarApp from "../../src";
import { ERROR_CODE } from "../../src/common";

const path = [44, 463, 5, 0, 3];

export default {
  name: "Ledger",
  props: {},
  data() {
    return {
      deviceLog: [],
      transportChoice: "WebUSB",
    };
  },
  computed: {
    ledgerStatus() {
      return this.deviceLog;
    },
  },
  methods: {
    log(msg) {
      this.deviceLog.push({
        index: this.deviceLog.length,
        msg,
      });
    },
    async getTransport() {
      let transport = null;

      this.log(`Trying to connect via ${this.transportChoice}...`);
      if (this.transportChoice === "WebUSB") {
        try {
          transport = await TransportWebUSB.create();
        } catch (e) {
          this.log(e);
        }
      }

      if (this.transportChoice === "U2F") {
        try {
          transport = await TransportU2F.create(10000);
        } catch (e) {
          this.log(e);
        }
      }

      return transport;
    },
    async getVersion() {
      this.deviceLog = [];

      // Given a transport (U2F/HIF/WebUSB) it is possible instantiate the app
      const transport = await this.getTransport();
      const app = new XarApp(transport);

      // now it is possible to access all commands in the app
      const response = await app.getVersion();
      if (response.return_code !== ERROR_CODE.NoError) {
        this.log(`Error [${response.return_code}] ${response.error_message}`);
        return;
      }

      this.log("Response received!");
      this.log(`App Version ${response.major}.${response.minor}.${response.patch}`);
      this.log(`Device Locked: ${response.device_locked}`);
      this.log(`Test mode: ${response.test_mode}`);
      this.log("Full response:");
      this.log(response);
    },
    async getPublicKey() {
      this.deviceLog = [];

      // Given a transport (U2F/HIF/WebUSB) it is possible instantiate the app
      const transport = await this.getTransport();
      const app = new XarApp(transport);

      let response = await app.getVersion();
      this.log(`App Version ${response.major}.${response.minor}.${response.patch}`);
      this.log(`Device Locked: ${response.device_locked}`);
      this.log(`Test mode: ${response.test_mode}`);

      // now it is possible to access all commands in the app
      response = await app.publicKey(path);
      if (response.return_code !== 0x9000) {
        this.log(`Error [${response.return_code}] ${response.error_message}`);
        return;
      }

      this.log("Response received!");
      this.log("Full response:");
      this.log(response);
    },
    async getAddress() {
      this.deviceLog = [];

      // Given a transport (U2F/HIF/WebUSB) it is possible instantiate the app
      const transport = await this.getTransport();
      const app = new XarApp(transport);

      let response = await app.getVersion();
      this.log(`App Version ${response.major}.${response.minor}.${response.patch}`);
      this.log(`Device Locked: ${response.device_locked}`);
      this.log(`Test mode: ${response.test_mode}`);

      // now it is possible to access all commands in the app
      response = await app.getAddressAndPubKey(path, "xar");
      if (response.return_code !== 0x9000) {
        this.log(`Error [${response.return_code}] ${response.error_message}`);
        return;
      }

      this.log("Response received!");
      this.log("Full response:");
      this.log(response);
    },
    async showAddress() {
      this.deviceLog = [];

      // Given a transport (U2F/HIF/WebUSB) it is possible instantiate the app
      const transport = await this.getTransport();
      const app = new XarApp(transport);

      let response = await app.getVersion();
      this.log(`App Version ${response.major}.${response.minor}.${response.patch}`);
      this.log(`Device Locked: ${response.device_locked}`);
      this.log(`Test mode: ${response.test_mode}`);

      // now it is possible to access all commands in the app
      this.log("Please click in the device");
      response = await app.showAddressAndPubKey(path, "xar");
      if (response.return_code !== ERROR_CODE.NoError) {
        this.log(`Error [${response.return_code}] ${response.error_message}`);
        return;
      }

      this.log("Response received!");
      this.log("Full response:");
      this.log(response);
    },
    async signExampleTx() {
      this.deviceLog = [];

      // Given a transport (U2F/HID/WebUSB) it is possible instantiate the app
      const transport = await this.getTransport();
      const app = new XarApp(transport);

      let response = await app.getVersion();
      this.log(`App Version ${response.major}.${response.minor}.${response.patch}`);
      this.log(`Device Locked: ${response.device_locked}`);
      this.log(`Test mode: ${response.test_mode}`);

      // now it is possible to access all commands in the app
      const message = String.raw`{"account_number":"6571","chain_id":"xarchain","fee":{"amount":[{"amount":"5000","denom":"uatom"}],"gas":"200000"},"memo":"some memo","msgs":[{"type":"csdt/MsgWithdrawCollateral","value":{"collateral_change":"7894","collateral_denom":"peanuts","sender":"xar1w34k53py5v5xyluazqpq65agyajavep27a5eqp"}}],"sequence":"0"}`;
      response = await app.sign(path, message);

      this.log("Response received!");
      this.log("Full response:");
      this.log(response);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

button {
  padding: 5px;
  font-weight: bold;
  font-size: medium;
}

ul {
  padding: 10px;
  text-align: left;
  alignment: left;
  list-style-type: none;
  background: black;
  font-weight: bold;
  color: greenyellow;
}
</style>
