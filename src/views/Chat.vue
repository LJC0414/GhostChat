<script setup lang="ts">
import dayjs from 'dayjs';
import '../assets/chat_style.css';
import 'https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.esm.js';
import 'https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.js';
import { nextTick, ref,  reactive } from 'vue'
import axios from 'axios'

import utc from 'dayjs/plugin/utc'
import tz from 'dayjs/plugin/timezone'
import router from '@/router';
dayjs.extend(utc);
dayjs.extend(tz);

const textSendT = ref([{
    text: '',
    userFrom: '',
    userFromId: 0,
    userTo: '',
    userToId: 0,
    time: '12:00'
}]);
const textSend = textSendT;
const text = ref('');
const user = ref({ headPortrait: 'friend1.jpg', name: 'a', id: 1 });
const userTo = ref({ name: 'b', id: 2 });
const friendsT = ref([{
    headPortrait: '1.jpg',
    name: 'a',
    id: 1,
    lastMessage: '1',
    lastTime: '12:00',
    status: 'bolck unread',
    unreadMessages: 0,
}]);
const friends = friendsT;
const searchId = ref('');
axios.post('/message', {
        params: {
            friendUserId: userTo.value.id,
        }
    }).then((res) => {
        console.log(res);
        textSend.value = res.data.success;
    }).catch((err) => {
        console.log(err);
    }
);

async function send() {
    if (text.value !== '') {
        // showTextSend();
        text.value = '';
        textSend.value.push({
            text: text.value,
            userFrom: user.value.name,
            userFromId: user.value.id,
            userTo: userTo.value.name,
            userToId: userTo.value.id,
            time: await dayjs().tz(await dayjs.tz.guess()).format('HH:mm'),
        });
        axios.post('/message', {
            text: text.value,
            userToID: userTo.value.id,
        }).then((res) => {
            console.log(res);
        }).catch((err) => {
            console.log(err);
        });

        axios.post('/message', {
                params: {
                    friendUserId: userTo.value.id,
                }
            }).then((res) => {
                console.log(res);
                textSend.value = res.data.success;
            }).catch((err) => {
                console.log(err);
            }
        );
        textSetToBottom();
    }
}

// function onInput(inputText: any) {
//     if (inputText.value.endsWith('\n')) {
//         text.value = '123123';

//         send();
//     }
// }

function textSetToBottom() {
    const area = document.getElementById('chatBox');
    if (area !== null) {
        nextTick(() => {
            area.scrollTop = area.scrollHeight;
        });
    }
}

async function changeTo(changeToId: number) {
    let checkInFriends = false;
    friends.value.forEach((i) => {
        if (i.id == changeToId) { //?????????????????????
            friends.value.forEach((j) => {
                if (j.id == userTo.value.id) {
                    j.status = 'block';
                }
            });
            i.status = 'block active';
            i.unreadMessages = 0;
            userTo.value.name = i.name;
            userTo.value.id = i.id;
            /* ??????????????????
            textSend.value = [
                {
                    text: 'hello ' + userTo.value.name,
                    userFrom: 'a',
                    userFromId: 1,
                    userTo: 'b',
                    userToId: 2,
                    time: '12:00'
                },
                {
                    text: '321321',
                    userFrom: 'b',
                    userFromId: 2,
                    userTo: 'a',
                    userToId: 1,
                    time: '12:01'
                },
                {
                    text: '123123123123123',
                    userFrom: 'a',
                    userFromId: 1,
                    userTo: 'b',
                    userToId: 2,
                    time: '12:02'
                },
                {
                    text: '321321dsafasd',
                    userFrom: 'b',
                    userFromId: 2,
                    userTo: 'a',
                    userToId: 1,
                    time: '12:05'
                },
            ];
             ?????????????????? */
            // Get messages from data base
            axios.post('/message', {
                    params: {
                        friendUserId: userTo.value.id,
                    }
                }).then((res) => {
                    console.log(res);
                    textSend.value = res.data.success;
                }).catch((err) => {
                    console.log(err);
                }
            );
            checkInFriends = true;
        }
    });
    if (!checkInFriends) { //???????????????????????????
        
    }
    // await freshTextArea();
}

function search() {
    // ??????????????????id ?????????????????????friends

}

function notSearching() {
    return searchId.value === null || searchId.value === '';
}

function quit() {
    router.push('../login');
}
</script>

<template>
    <div class="bodyStyle">
        <div class = "box">
            <div class = "leftSide">
                <div class = "header">
                    <div class = "userpic">
                        <!-- ??????????????? -->
                        <img :src = "user.headPortrait" class = "cover">
                    </div>
                    <div class = "userName">
                        <!-- ??????????????? -->
                        <h4>{{ user.name }}<br><span>uid:{{ user.id }}</span></h4>
                    </div>
                    <ul class = "header_icons">
                        <!--  https://ionic.io/ionicons  -->
                        <li><ion-icon @click="quit" name="log-out-outline"></ion-icon></li>
                        <li><ion-icon name="chatbubble-ellipses-outline"></ion-icon></li>
                        <li><ion-icon name="archive-outline"></ion-icon></li>
                    </ul>
                </div>
                <!-- Search -->
                <div class = "searchBar">
                    <div>
                        <input type = "text" placeholder="search for uid or username" v-model="searchId" @input="search">
                        <ion-icon name="search-outline"></ion-icon>
                    </div>
                </div>
                <!-- search list // history chat list -->
                <div class = "chatlist">
                    <div v-if="notSearching()">
                        <div  v-for="friend in friends" :class = "friend.status" @click="changeTo(friend.id)">
                            <div class = "imgbk">
                                <!-- ?????? -->
                                <img :src = "friend.headPortrait" class = "cover">
                            </div>
                            <div class = "infors">
                                <div class = "inforHead">
                                    <!-- ??????????????? -->
                                    <h4>{{ friend.name }}</h4>
                                    <!-- ?????? -->
                                    <p class = "time">{{ friend.lastTime }}</p>
                                </div>
                                <div class = "message_list">
                                    <!-- ?????? -->
                                    <p>{{ friend.lastMessage }}</p>
                                    <b v-if="friend.unreadMessages!==0">{{ friend.unreadMessages }}</b>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div v-else>
                        <div  v-for="friend in friends" :class = "friend.status" @click="changeTo(friend.id)">
                            <div class = "imgbk">
                                <!-- ?????? -->
                                <img :src = "friend.headPortrait" class = "cover">
                            </div>
                            <div class = "infors">
                                <div class = "inforHead">
                                    <!-- ??????????????? -->
                                    <h4>{{ friend.name }}</h4>
                                </div>
                                <div class = "message_list">
                                    <!-- ?????? -->
                                    <p>uid: {{ friend.id }}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class = "rightSide">
                <div class = "header">
                    <div class = "chatName">
                        <!-- ????????????????????? -->
                        <h4>{{ userTo.name }}</h4>
                    </div>
                    <ul class = "header_icons">
                        <!--  https://ionic.io/ionicons  -->
                        <li><ion-icon name="happy-outline"></ion-icon></li>
                        <li><ion-icon name="add-circle-outline"></ion-icon></li>
                        <li><ion-icon name="folder-outline"></ion-icon></li>
                    </ul>
                </div>

                <div class = "chatBox" id="chatBox">
                    <a href = "./forgot.html">history messages</a>
                    <div v-for="message in textSend" :class = "message.userFrom===user.name?'message self':'message other'">
                        <p>{{ message.text }}<br><span>{{ message.time }}</span></p>
                    </div>
                </div>

                <div class = "chatBox-input">
                    <ion-icon name="beer-outline"></ion-icon>
                    <ion-icon name="bed-outline"></ion-icon>
                    <input type = "text" placeholder= "Type something here" v-model="text">
                    <ion-icon name="send-outline" @click="send"></ion-icon>
                </div>

            </div>
        </div>
    </div>
</template>