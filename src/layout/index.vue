<template>
  <div class="Layout">
    <div class="TopBar">
      <div class="logo" @click="back">
        <!-- <p class="p1">NFT Lease</p>
        <p class="p2">NFT Lease</p> -->
        <img src="../assets/img/public/logo.svg" />
      </div>
      <div class="search-area">
        <div class="input">
          <n-icon class="icon" size="18">
            <Search />
          </n-icon>
          <input
            placeholder="Search: Account ID"
            @keyup.enter="search"
            v-model="searchAccountId"
          />
        </div>
      </div>
      <div class="tool-btns">
        <button class="connect-btn" @click="nearAccount">
          {{ gitAccountId ? gitAccountId : "connect-wallet" }}
        </button>
        <button
          v-if="gitAccountId"
          class="connect-btn signOut"
          @click="signOut"
        >
          Sign out
        </button>
        <!-- <n-switch
          class="toggle-lang-btn"
          :value="lang"
          checked-value="zh"
          unchecked-value="en"
          @update:value="toggle_lang"
        >
          <template #checked>EN</template>
          <template #unchecked>中文</template>
        </n-switch> -->
      </div>
    </div>
    <div class="bg-wrap" @scroll="scroll">
      <div class="container">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>
import { getCurrentInstance, ref } from "vue";
import { mapActions } from "vuex";
import { useMessage } from "naive-ui";
import { useI18n } from "vue-i18n";
import bus from "vue3-eventbus";
import {
  GameControllerOutline,
  GameController,
  Search,
} from "@vicons/ionicons5";
import { useRouter } from "vue-router";

export default {
  name: "Layout",
  components: {
    GameController,
    GameControllerOutline,
    Search,
  },
  data() {
    return {
      accountId: null,
    };
  },
  methods: {
    ...mapActions(["update"]),
    async nearAccount() {
      if (this.gitAccountId) {
        // this.$near.walletConnection.signOut();
        // window.location.replace(
        //   window.location.origin + window.location.pathname
        // );
        window.open("https://wallet.testnet.near.org/");
      } else {
        await this.$near.loginAccount();
      }
    },
    signOut() {
      this.$near.walletConnection.signOut();
      window.location.replace(
        window.location.origin + window.location.pathname
      );
    },
    setAccount(name) {
      this.accountId =
        this.$near.user && this.$near.user.accountId
          ? this.$near.user.accountId
          : null;
      this.update({ key: "account_id", value: this.accountId });
      this.update({ key: "account", value: { ...this.$near.user } });
    },
    scroll(e) {
      if (
        e.srcElement.scrollTop + e.srcElement.offsetHeight >
        e.srcElement.scrollHeight - 1
      ) {
        switch (this.$route.path) {
          case "/home/browse":
            window.nexPageBrowse();
            break;
          case "/home/collectible":
            window.nexPageCollectible();
            break;
          default:
            break;
        }
      }
      console.log();
    },
  },
  mounted() {
    setTimeout(async () => {
      this.setAccount();
      //余额
      // let account = await this.$near.near.account(this.$near.user.accountId)
      // console.log(await account.state() );
      // console.log(this.$near.config);
    }, 40);
    setTimeout(async () => {
      this.setAccount();
    }, 4000);
  },
  computed: {
    gitAccountId() {
      return this.$store.getters.account_id;
    },
  },
  setup() {
    const message = useMessage();
    const { locale } = useI18n();
    const lang = ref(localStorage.getItem("lang"));
    const { proxy } = getCurrentInstance();
    const router = useRouter();
    const searchAccountId = ref(""); //搜索栏用户id
    // 搜索用户id
    const search = async () => {
      console.log(searchAccountId);
      if (searchAccountId) {
        try {
          const tokens = await proxy.useParasApi("nft_tokens_for_owner", {
            account_id: searchAccountId.value,
          });
          bus.emit("search", { tokens });
          router.push({
            name: "Search",
            query: {
              id: searchAccountId.value,
              // data: JSON.stringify(tokens),
            },
          });
          console.log(tokens);
        } catch (error) {}
      }
    };

    const back = () => {
      router.push("/home");
    };
    return {
      toggle_lang(value) {
        lang.value = value;
        locale.value = value;
        localStorage.setItem("lang", value);
        message.info(value);
      },
      lang,
      // 搜索
      search,
      searchAccountId,
      back,
    };
  },
};
</script>

<style lang="scss" scoped>
.Layout {
  display: flex;
  height: 100%;
  flex-direction: column;
  min-height: 0;
  flex: 1 1;
}
.TopBar {
  background: white;
  width: 1344px;
  height: 60px;
  display: flex;
  justify-content: space-between;
  margin: 0 auto;
  align-items: center;
  @media screen and (max-width: 720px) {
    width: 100%;
    flex-direction: column;
    padding-top: 10px;
    padding-bottom: 10px;
    box-sizing: border-box;
    height: auto;
    & > .search-area {
      width: 90% !important;
      margin: 10px auto;
    }
    // .tool-btns{
    //   display: none !important;
    // }
  }
  & > div {
    display: flex;
    justify-content: center;
  }
  .logo {
    cursor: pointer;
    width: fit-content;
    margin-right: 10px;
    font-size: 30px;
    position: relative;
    letter-spacing: 2px;
    .p1 {
      color: white;
      z-index: 3;
    }
    .p2 {
      position: absolute;
      top: 0;
      color: #ffac32;
      z-index: 2;
      transform: translate(3px, 3px);
    }
  }
  .search-area {
    width: 0;
    flex: 1;
    display: flex;
    align-items: center;

    .input {
      position: relative;
      width: 938px;
      height: 34px;

      .icon {
        position: absolute;
        color: #fde47c;
        left: 8px;
        top: 10px;
      }
      input {
        width: 100%;
        box-sizing: border-box;
        width: 100%;
        height: 100%;
        border: 1px solid #fde47c;
        border-radius: 10px;
        outline: none;
        padding-left: 30px;
        &:focus-visible {
          border: 1px solid #fde47c;
        }
      }
    }
  }
  .tool-btns {
    width: 217px;
    display: flex;
    align-items: center;
    margin-left: 10px;
    .connect-btn {
      font-family: "Barlow", sans-serif;
      cursor: pointer;
      width: 140px;
      height: 34px;
      background: #fde47c;
      border: 1px solid #fdcc01;
      border-radius: 10px;
      font-weight: 600;
      font-size: 14px;
      line-height: 14px;
      text-align: center;
      color: #000000;
    }
    .signOut {
      margin-left: 10px;
      background: #ec1a1a;
      border: 1px solid #fd0101;
      color: #fff;
    }
    .toggle-lang-btn {
      margin-left: 10px;
    }
  }
}
.bg-wrap {
  flex: 1 1;
  overflow-y: scroll;
  background-color: #fff9e7;
  .container {
    width: 1344px;
    margin: 0 auto;
  }
}
</style>