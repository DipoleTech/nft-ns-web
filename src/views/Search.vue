<template>
  <n-spin :show="loading">
    <div class="card-group">
      <div
        class="card-wrap"
        @click="() => detail(index)"
        v-for="(item, index) in collectibles.values"
        :key="index"
      >
        <div class="card">
          <div class="top">
            <img :src="item.img" />
          </div>
          <div class="bottom">
            <button>Bid Now</button>
          </div>
        </div>
      </div>
      <div class="empty" v-if="collectibles.values.length === 0">
        <img src="../assets/img/public/no-nft.png" />
      </div>
    </div>
  </n-spin>
</template>

<script>
import { mapActions } from "vuex";
import { reactive, ref } from "@vue/reactivity";
import { useRouter, useRoute } from "vue-router";
// import chainMixin from "../utils/chainMixin";
import { onMounted } from "vue";
import { getCurrentInstance } from "vue";
import bus from "vue3-eventbus";
export default {
  name: "Search",

  setup() {
    const router = useRouter();

    const { proxy } = getCurrentInstance();
    const tokens = reactive([]);
    const collectibles = reactive([]);
    const loading = ref(true);
    const route = useRoute();

    bus.on("search", (e) => {
      // console.log("on seach");
      // render(e.tokens);
      setTimeout(() => {
        console.log(route.query);
        get_data(route.query.id);
      }, 40);
    });
    const render = (_tokens) => {
      const media_base_url = "https://ipfs.fleek.co/ipfs/";
      tokens.values = _tokens;
      loading.value = false;
      collectibles.values = tokens.values.map((e) => {
        return {
          img: media_base_url + e.metadata.media,
          title: e.metadata.title,
          data: e,
        };
      });
    };
    const detail = (index) => {
      const { token_id } = tokens.values[index];
      router.push("/detail/" + token_id);
    };

    const get_data = async (id) => {
      loading.value = true;

      console.log(id);
      const _tokens = await proxy.useParasApi("nft_tokens_for_owner", {
        account_id: id,
      });
      loading.value = false;
      render(_tokens);
      // console.log(tokens);
    };
    onMounted(() => {
      // search by route
      setTimeout(() => {
        console.log(route.query);
        if (route.query.id) {
          get_data(route.query.id);
        }
      }, 40);
      // if (route.params.data) {
      //   console.log(route.params);
      //   render(JSON.parse(route.params.data));
      // } else {
      // }
    });
    return {
      detail,
      loading,
      collectibles,
      tokens,
    };
  },
};
</script>
<style lang="scss" scoped>
.card-group {
  min-height: 2 * 385px;

  max-height: 2 * 385px;
  overflow-y: scroll;
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
      height: 325.63px;
      overflow: hidden;
      box-sizing: border-box;
      padding: 16px;
      transition: 0.3s;
      margin-top: 16px;
      transform: translateY(0);
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
        img {
          width: 100%;
          height: 100%;
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
    margin-top: 200px;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    img {
      width: 147px;
    }
  }
}
</style>
