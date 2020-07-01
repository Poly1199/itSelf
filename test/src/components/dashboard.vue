<template>
 <div>
     <div v-for="item in channelsStatistics" :key="item.channelName">
         <h2><b>Channal name: {{item.channelName}}</b></h2>
         <p>total count: {{item.totalCount}}</p>
         <div v-for="(contentType,name) in item.contentType" :key="name">
             contentType: {{name}} count: {{contentType.count}}
             sessionDuration: {{contentType.time}}
             perennial contentType use: {{contentType.count/(item.totalCount/100)}} %
         </div>
     </div>
 </div>
</template>

<script>
import axios from 'axios';
import  * as csv from 'csvtojson';

export default {
    name: 'Dashboard',
    data() {
        return {
            info: {},
            statistic: [],
            tvChannals: [],
            channelsStatistics: [],
        }
    },
    mounted () {
        axios.get('https://provisioning-test.itself.cz/api/homework/service/2/viewerstat/2020-06-25', {
                auth: {
                    username:'applicant8',
                    password:'BevJalorwud5'
                }
            })
            .then(async (response) => {
                this.info = response;
                const dataCsv = response.data;
                await csv({
                    headers: ["subscriptionId", "clientIp" ,"contentType" ,"channelName" ,"sessionStart" ,"sessionEnd" ,"sessionDuration" ,"sessionLength" ,"bandwidth" ,"clientUa"]
                }).fromString(dataCsv).then((csvRow) => this.statistic= csvRow);
                this.getTvChannelNames();
                this.getContentType();
            })

    },
    methods: {
      getTvChannelNames() {
          this.statistic.forEach((el) => {
              if(this.tvChannals.indexOf(el.channelName) === -1) {
                  this.tvChannals.push(el.channelName);
              }
          });
          this.channelsStatistics = this.tvChannals.map(channelName => ({ channelName, totalCount: 0, contentType: {} }));
      },
      getContentType() {
          this.statistic.forEach((el) => {
              const record = this.channelsStatistics.find((channel) => channel.channelName === el.channelName);
              if (!record.contentType[el.contentType]){
                  record.contentType[el.contentType] = {count: 0, time: "00:00:00"}
              }
              record.contentType[el.contentType].count ++;
              const splitLast = (record.contentType[el.contentType].time).split(":");
              const splitAdded = (el.sessionDuration).split(":");
              const lastTimeSeconds = (+splitLast[0]) * 60 * 60 + (+splitLast[1]) * 60 + (+splitLast[2]);
              const addedTimeSeconds = (+splitAdded[0]) * 60 * 60 + (+splitAdded[1]) * 60 + (+splitAdded[2]);
              const date = new Date(1970,0,1);
              date.setSeconds(lastTimeSeconds + addedTimeSeconds);
              record.contentType[el.contentType].time = date.toTimeString().replace(/.*(\d{2}:\d{2}:\d{2}).*/, "$1");
              record.totalCount++;
          })
      }
    },
}
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
</style>
