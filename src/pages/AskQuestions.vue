<template>
    <div id="app">
        <h4 style="color: white; margin-top: 10px; margin-left: 10px" >當前查詢：</h4>
        <h4 style="color: white; margin-top: 10px; margin-left: 10px">{{this.$route.query.Name}} （{{this.$route.query.companyId}})</h4>
        <h5 style="color: white; margin-top: 10px; margin-left: 10px">{{this.$route.query.Year}},{{this.$route.query.Season}}</h5>
      <AskQuestions id = "AskQuestions"
        iconColorProp="#e6e6e6"
        messageOutColorProp="#4d9e93"
        messageInColorProp="#f1f0f0"
        :messageListProp="messageList"
        :initOpenProp="initOpen"
        @onToggleOpen="handleToggleOpen"
        @onMessageWasSent="handleMessageReceived"
      />


    </div>
</template>

<script>
import AskQuestions from '../components/Self/ChatWidget.vue';
import incomingMessageSound from '../assets/pop.mp3' // pick an audio file for chat response

export default {
    name: 'askQuestions',
    components: {
        AskQuestions,
    },
    data: () => ({
        messageList: [],
        initOpen: true,
        toggledOpen: false,
    }),
    // 剛在入頁面就會執行 看一下有沒有正確帶參數(其實url也會掛Q)
    mounted() {
        console.log('=============== test params ==============');
        console.log(this.$route.query.companyId);
        console.log(this.$route.query.Name);
        console.log(this.$route.query.Year);
        console.log(this.$route.query.Season);
        console.log('=============== test params ==============');
        this.messageList.push({ body: '歡迎提出您的問題', author: 'them' });
    },
    methods: {
    // Send message from you
        // 送出訊息 要調成跟JsonBerting要得參數
        handleMessageReceived(message) {
            this.messageList.push(message);
            let cxt = '';
            
            let query ='';

            this.axios({
                method: 'post',
                url: 'http://127.0.0.1:5040/embedding',
                data: {
                    query: message.body,
                    year: this.$route.query.Year,
                    season: this.$route.query.Season,
                    company: this.$route.query.companyId,
                },
            })
                .then((response) => {
                    cxt = response.data;
                    this.axios({
                                method: 'post',
                                url: 'http://127.0.0.1:5050/qa',
                                data: { query: message.body, context: cxt },
                    })
                        .then((res) => {
                            this.messageList.push({ body: res.data, author: 'them' });
                            var messageDisplay = document.getElementById("ChatArea");
                            var AskQuestions = document.getElementById("AskQuestions");
                            this.handleMessageResponseSound();
                            
                                messageDisplay.scrollTop = messageDisplay.scrollHeight;
                           
                            console.log('================messageDisplay');
                            console.log(messageDisplay);
                            console.log(messageDisplay.scrollTop);
                            console.log(messageDisplay.scrollHeight);
                            messageDisplay.scrollTop = messageDisplay.scrollHeight;
                            this.axios({
                                method: 'post',
                                url: 'http://127.0.0.1:5020/record',
                                data: { 
                                    query: message.body,
                                    answer: res.data,
                                    year: this.$route.query.Year,
                                    season: this.$route.query.Season,
                                    user_id: 2,
                                    company_id: this.$route.query.companyId
                                },
                            }).then((back) => {
                                
                            }).catch((error) => {
                                
                            });
                        })
                        .catch((error) => {
                            // eslint-disable-next-line
                            console.log(error);
                        });
                })
                .catch((error) => {
                    // eslint-disable-next-line
                    console.log(error);
                });
            
},
        // Receive message from them (handled by you with your backend)
        handleMessageResponse(message) {
          
                this.messageList.push({ body: '嗨~~~', author: 'them' });
            
        },
        // Chat toggled open event emitted
        handleToggleOpen(open) {
            this.toggledOpen = open;
            // connect/disconnect websocket or something
        },
        handleMessageResponseSound() {
            const audio = new Audio(incomingMessageSound)
            audio.addEventListener('loadeddata', () => {
            audio.play()
            })
        },
    // Audible chat response noise, use whatever noise you want
    },
    // init chat with a message
    watch: {
        messageList(newList) {
            const nextMessage = newList[newList.length - 1];
            const isIncoming = (nextMessage || {}).author !== 'you';
            if (isIncoming && this.toggledOpen) {
                this.handleMessageResponseSound();
            }
        },
    },
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
</style>
