<template>
  <div class="chat">
    <br />
    <h1>ODB Chat Test</h1>
    <div id="chatwindow">
      <div id="chat">
        <div><h4>Messages from beyond time and space:</h4></div>
        <p v-for="msg in outerMessages">{{ msg.name }} says: {{ msg.msg }}</p>
        <div v-if="ipfsDBREADY">
          <h3>Connected DB:</h3>
          <p>{{ address }}</p>
        </div>
      </div>
      <div id="registerwindow">
        <div v-if="!ipfsDBREADY">
          <label
            >Open Existing DB Hash: <input type="text" v-model="userDbHash"
          /></label>
          <input type="button" @click="connectExisting" value="Connect" />
          <br /><br />
          <button @click="connectNew">Create New DB</button>

          <input class="input2" v-model="nickName" />
          <input class="input1" v-model="orbitPackage" />
          <input type="button" @click="launchToOrbit" value="send message" />
        </div>
      </div>
      <!-- <div>{{ outerMessages }}</div> -->
    </div>
  </div>
</template>

<script>
import OrbitDB from "orbit-db";
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data: function() {
    return {
      ipfs: null,
      userDbHash: null,
      ipfsDBREADY: false,
      orbitdb: null,
      db: null,
      address: null,
      nickName: "username" + Math.floor(Math.random() * 100),
      orbitPackage: null,
      outerMessages: null
    };
  },
  mounted: function() {
    this.ipfsOptions = {
      EXPERIMENTAL: {
        pubsub: true
      },
      config: {
        Addresses: {
          Swarm: [
            // Use IPFS dev signal server
            // '/dns4/star-signal.cloud.ipfs.team/wss/p2p-webrtc-star',
            "/dns4/ws-star.discovery.libp2p.io/tcp/443/wss/p2p-websocket-star"
            // Use local signal server
            // '/ip4/0.0.0.0/tcp/9090/wss/p2p-webrtc-star',
          ]
        }
      }
    };

    //this.ipfs = new IPFS(ipfsOptions)
  },
  created: function() {
    console.log("created");
  },
  computed: {},
  methods: {
    connectNew: function() {
      this.ipfs = new window.Ipfs(this.ipfsOptions);
      this.ipfs.on("ready", async () => {
        this.orbitdb = new OrbitDB(this.ipfs);
        const access = {
          // Give write access to ourselves
          //write: [this.orbitdb.key.getPublic("hex")]
          write: true ? ["*"] : []
        };

        this.db = await this.orbitdb.feed("flashT3st", access);
        this.address = this.db.address.toString();
        console.log("orbitdb ready: " + this.address);

        //this.db = await this.orbitdb.feed('zennifyMeDBS00PER1337')
        this.ipfsDBREADY = true;
        this.collectMessagesOnTimer();
      });
    },
    connectExisting: function() {
      console.log("Connect Existing DB: " + this.userDbHash);
      this.ipfs = new window.Ipfs(this.ipfsOptions);
      this.ipfs.on("ready", async () => {
        this.orbitdb = new OrbitDB(this.ipfs);
        const access = {
          // Give write access to ourselves
          write: [this.orbitdb.key.getPublic("hex")]
        };

        //this.db = await this.orbitdb.feed("flashT3st", access);
        this.db = await this.orbitdb.open(this.userDbHash);
        this.address = this.db.address.toString();
        console.log("orbitdb ready: " + this.address);

        //this.db = await this.orbitdb.feed('zennifyMeDBS00PER1337')
        this.ipfsDBREADY = true;
        this.collectMessagesOnTimer();
      });
    },

    launchToOrbit: function() {
      console.log("inside launchToOrbit");
      console.log("value of this.ipfs: ", this.ipfs);
      if (this.ipfsDBREADY === true) {
        console.log("inside launchToOrbit handler");
        console.log("value of this.db: ", this.db);
        const asyncCall = async () => {
          const hash = await this.db.add({
            name: this.nickName,
            msg: this.orbitPackage
          });
          console.log("value of the hash: ", hash);
          this.outerMessages = await this.db
            .iterator({ limit: -1 })
            .collect()
            .map(e => e.payload.value);
        };
        asyncCall();
      }
    },
    collectMessagesOnTimer: function() {
      console.log("inside collectMessagesOnTimer");
      const timerFunc = () => {
        const asyncCall = async () => {
          console.log("collecting messages");
          this.outerMessages = await this.db
            .iterator({ limit: -1 })
            .collect()
            .map(e => e.payload.value);
        };
        asyncCall();
        setTimeout(() => {
          timerFunc();
        }, 2500);
      };
      timerFunc();
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.chat {
}
#chatwindow {
  bottom: 0;
  width: 500px;
  height: 500px;
  margin: auto;
  border-radius: 15px;
  background-color: grey;
}
#registerwindow {
  background-color: white;
  height: 700px;
  border-radius: 15px;
  background-color: aqua;
  padding-top: 500px;
}
#chat {
  position: absolute;
  top: 150px;
  width: 500px;
  height: 500px;
  bottom: 0;
  overflow-y: scroll;
}
</style>
