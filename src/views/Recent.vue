<template>
  <n-spin :show="loading">
    <div class="card-group">
      <div
        class="card-wrap"
        v-for="(item, index) in imgs"
        :key="index"
        @click="detail(index)"
      >
        <div :class="'card ' + (item.expired ? 'Expired' : item.bid_state)">
          <div class="top">
            <div :class="'mask '">
              <img
                v-if="item.expired || item.bid_state === 'Rejected'"
                src="../assets/img/public/expired.png"
              />
              <img
                v-else-if="item.bid_state === 'InProgress'"
                src="../assets/img/public/inprogress.png"
              />
            </div>
            <img :src="item.metadata.img" />
          </div>
          <!-- <div class="bottom"> -->
          <!-- <button @click="lease(item.data)">Bid Now</button> -->
          <!-- <button>Bid Again</button> -->
          <!-- </div> -->
        </div>
      </div>
      <div class="empty" v-if="imgs.length === 0">
        <img src="../assets/img/public/no-nft.png" />
      </div>
    </div>
  </n-spin>
</template>

<script>
import loading from "naive-ui/lib/_internal/loading";
export default {
  data: () => {
    return {
      imgs: [],
      loading: true,
    };
  },
  methods: {
    detail(index) {
      // console.log(this.imgs[index]);
      // const { token_id } = tokens.values[index];
      this.$router.push("/detail/" + this.imgs[index].token_id);
    },
  },
  mounted() {
    setTimeout(async () => {
      let bidData = await this.useNnsApi("list_bids_by_sender", {
        sender_id: this.$store.getters.account_id,
      });

      let nftData = [];
      // 循环报价历史
      for (const key in bidData) {
        let token_id_meta = bidData[key].src_nft_id.split(":");
        let data = {
          ...bidData[key],
          bid_id: key,
          metadata: {},
          token_id: token_id_meta[1] + ":" + token_id_meta[2],
          expired:
            (bidData[key].start_at + bidData[key].lasts) * 1000 <
            new Date().getTime(),
        };
        nftData.push(data);
        // 循环nft数据信息
        for (let index = 0; index < nftData.length; index++) {
          // 如果在此之前有相同的token_id数据就将其metadata拷贝至末位，如果没有就向链侧获取数据
          if (index === nftData.length - 1) {
            let data = await this.useParasApi("nft_token", {
              token_id: nftData[index].token_id,
            });
            nftData[index].metadata = data.metadata;
            nftData[index].metadata.img =
              "https://ipfs.fleek.co/ipfs/" + data.metadata.media;
          } else if (
            nftData[index].token_id === nftData[nftData.length - 1].token_id
          ) {
            nftData[nftData.length - 1].metadata = nftData[index].metadata;
            break;
          }
        }
      }
      this.imgs = nftData;
      console.log(this.imgs);
      this.loading = false;
    }, 40);
    // setTimeout(async () => {
    // const tokens = await this.useApi('nft_tokens_for_owner',{account_id: 'huishanlhr2.testnet' })
    // // 拼接url
    // // this.loading=false
    // const media_base_url = "https://ipfs.fleek.co/ipfs/";
    // this.imgs = tokens.map((e) => ({
    //   img: media_base_url + e.metadata.media,
    //   title: e.metadata.title,
    //   data: e
    // }));
    // }, 40);
  },
};
</script>
<style lang="scss" scoped>
.card-group {
  min-height: 385px;
  margin-top: 32px;
  padding: 19px;
  box-sizing: border-box;
  background-color: #fff5d6;
  border-radius: 20px;
  .card-wrap {
    width: 20%;
    display: inline-block;
    .card {
      border: 1px solid rgba(245, 205, 146, 0.7);
      box-sizing: border-box;
      border-radius: 10px;
      width: 232px;
      // height: 325.63px;
      overflow: hidden;
      box-sizing: border-box;
      padding: 16px;
      transition: 0.3s;
      margin-top: 16px;
      transform: translateY(0);
      // 等待响应
      &.InProgress,
      &.Expired,
      &.Rejected {
        .top .mask {
          display: flex;
        }
      }
      &:hover {
        cursor: pointer;
        transform: translateY(-3px);

        // box-shadow: 0px 0px 10px rgb(116, 116, 116);
        box-shadow: 8px 8px 11px #f0e6c9, -8px -8px 11px #ffffe3;
      }
      .top {
        width: 100%;
        height: 250px;
        overflow: hidden;
        border-radius: 10px;
        position: relative;

        .mask {
          width: 100%;
          height: 100%;
          position: absolute;
          top: 0;
          left: 0;
          background-color: rgba(0, 0, 0, 0.6); //出价了
          display: none;
          align-items: center;
          justify-content: center;
          img {
            width: 40px;
            height: 40px;
          }
        }
        img {
          height: 100%;
          width: 100%;
          object-fit: cover;
        }
      }
      .bottom {
        margin-top: 15px;
        height: 32px;
        font-weight: 600;
        text-align: center;
        button {
          background: #fecc00;
          border-radius: 6px;
          width: 200px;
          height: 32px;
          border: none;
          font-family: Barlow;
          font-style: normal;
          font-weight: 600;
          font-size: 15px;
          color: #000000;
        }
      }
    }
  }
  .empty {
    margin-top: 100px;
    display: flex;
    width: 100%;
    justify-content: center;
    align-items: center;
    height: 100%;
    text-align: center;
    img {
      width: 140px;
    }
  }
}
</style>
