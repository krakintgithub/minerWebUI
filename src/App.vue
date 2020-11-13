<template>


  <div class="hasmetamask">
    <div class="popup" id="popup">
      <div class="popup-inner">


        <div class="miner">
          <img src="./assets/img/miner.png" alt="" width="100%" class="miner">
        </div>
      </div>

      <!--        <div class="install-metamask">Please install:<br><img src="./assets/img/metamask_logo.png" height="50px"><br>-->
      <!--          Metamask.<br> Then, reload...</div>-->

      <div class="refreshButton" v-on:click="reloadMiner()"
           onmouseenter="displreload()" onmouseleave="mouseExit()"
      ></div>



      <div class="depscr" style="display: none">DEPOSIT<br><img src="./assets/img/deposit.png" height="50px">
        <div style="text-align: left">
          <li>Enter amount</li>
          <li>Click green button</li>
        </div>
      </div>

      <div class="withscr" style="display: none">WITHDRAW<br><img src="./assets/img/withdraw.png" height="50px">
        <div style="text-align: left">
          <li>Enter amount</li>
          <li>Click red button</li>
        </div>
      </div>

      <div class="reloadscr" style="display: none">RELOAD<br><img src="./assets/img/reload.png" height="50px">
        <div style="text-align: left">
          <li>Connect to Metamask</li>
          <li>Press Reload</li>
        </div>
      </div>


      <!--        <div class="metamask-link"><a href="https://metamask.io/download.html" target="_blank">Click Here</a></div>-->
      <div class="balance">
        Miner Balance: <br>
        <div id="balance_num" style="display: inline-block;">Connect to Metamask</div>
        <br><br>
        Block Number: <br>
        <div id="block_num" style="display: inline-block;">Connect to Metamask</div>
      </div>
      <div class="txtform"><input type="text" id="tbox" name="tbox" placeholder="Enter amount"></div>
      <div class="depbttn" id="depbttn" onmouseenter="displdep()" onmouseleave="mouseExit()">
        <div class="container">
          <b-button
              @click="processDeposit"
              class="depositButton"
          >

          </b-button>
        </div>
      </div>
      <div class="wthbttn" id="wthbttn" onmouseenter="displwtn()" onmouseleave="mouseExit()">
        <div class="container">
          <b-button
              @click="processWithdraw"
              class="withdrawButton"
          >
          </b-button>
        </div>
      </div>

    </div>
  </div>


</template>

<script>
import web3 from '../contracts/web3';
import auctionBox from '../contracts/auctionBoxInstance';

function numberWithCommas(x) {
  return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

function truncateNumber(x) {
  return Math.trunc(x * 1000) / 1000;
}

function formatNumber(x) {
  return x.replace(/,/g, '');
}

function formatAddress(address) {
  return address.substr(0, 5) + "..." + address.substr(35);
}


export default {
  name: 'APP',
  created: function () {
    this.interval = setInterval(() => this.refreshData(), 3000);
  },
  data: function () {
    return {
      userAddress: 'Connect to Metamask, then reload.',
      miners: '-1',
      totalBurned: '-1',
      totalMinted: '-1',
      blockNumber: '-1',
      reward: 'Connect to Metamask',
      deposit: '',
      withdraw: '',
    };
  },
  beforeMount() {
    this.refreshData();
  },
  methods: {
    reloadMiner(){window.location.reload(true);},
    refreshData() {
      this.userAddress = formatAddress(web3.eth.accounts.givenProvider.selectedAddress);
      const fromAddress = web3.eth.accounts.givenProvider.selectedAddress;

      auctionBox.methods
          .getCurrentBlockNumber()
          .call()
          .then((n) => {
            this.blockNumber = n;
            document.getElementById("block_num").innerText = this.blockNumber;
          });
      auctionBox.methods
          .showReward(fromAddress)
          .call()
          .then((n) => {
            this.reward = numberWithCommas(truncateNumber(web3.utils.fromWei(n, 'ether')));
            document.getElementById("balance_num").innerText = this.reward;
          });
    },

    processDeposit() {
      const fromAddress = web3.eth.accounts.givenProvider.selectedAddress;
      const tboxval = document.getElementById('tbox').value;
      const amount = web3.utils.toWei(formatNumber(tboxval), 'ether');
      this.deposit = '';

      auctionBox.methods
          .mine(amount).send({
        from: fromAddress,
      }).then(() => {
        document.getElementById('tbox').value = '';
      });
    },
    processWithdraw() {
      const fromAddress = web3.eth.accounts.givenProvider.selectedAddress;
      const tboxval = document.getElementById('tbox').value;
      const amount = web3.utils.toWei(formatNumber(tboxval), 'ether');
      this.withdraw = '';

      auctionBox.methods
          .getReward(amount).send({
        from: fromAddress,
      }).then(() => {
        document.getElementById('tbox').value = '';
      });

    }

  },
};
</script>

<style>

#tbox{
  width: 160px !important;
}

.depositButton {
  margin-top: 3px;
  margin-right: 26px !important;
  margin-left: -13px;
  border-radius: 100px !important;
  height: 70px;
  width: 69px;
  background: none !important;
  border: 0px !important;
  cursor: pointer;
}

.withdrawButton {
  border: 0px !important;
  margin-top: 3px;
  margin-left: -13px;
  border-radius: 100px !important;
  height: 69px;
  width: 69px;
  background: none !important;
  cursor: pointer;
}


.refreshButton {
  border: 0px !important;
  border-radius: 100px !important;
  height: 46px;
  width: 46px;
  background: none !important;
  position: absolute;
  margin-top: -466px;
  margin-left: 393px;
  cursor: pointer;
}



</style>
