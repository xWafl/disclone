<template>
    <div class="loginPage">
        <h1 class="welcomeMsg">Welcome to Aquifer</h1>
        <div
            class="loginBox"
        >
            <div
                class="loginHeaders"
            >
                <LoginHeader
                    :id="0"
                    :selected="selectedNum"
                    @changeSelected="changeSelected('login', 0)"
                    name="Login"
                ></LoginHeader>
                <LoginHeader
                    :id="1"
                    :selected="selectedNum"
                    @changeSelected="changeSelected('create', 1)"
                    name="Create"
                ></LoginHeader>
                <LoginHeader
                    :id="2"
                    :selected="selectedNum"
                    @changeSelected="changeSelected('logout', 2)"
                    name="Logout"
                ></LoginHeader>
            </div>
            <div class="usernameInputs">
                <input
                    v-if="selected !== 'logout'"
                    :style="[selected === 'login' ? {'width': '60%'} : {'width': '90%'}]"
                    class="loginUsername"
                    v-model="username" type="text"
                    placeholder="Username..."
                />
                <input
                    v-if="selected === 'login'"
                    class="loginUsernum"
                    v-model="usernum"
                    type="text"
                    placeholder="Usernum..."
                />
            </div>
            <input
                v-if="selected !== 'logout'"
                class="loginInput"
                v-model="password"
                type="password"
                placeholder="Password..."
            />
            <div
                v-if="selected === 'login'"
                class="loginSubmit"
                v-on:click="loginUser"
            >Log in</div>
            <div
                v-if="selected === 'create'"
                class="loginSubmit"
                v-on:click="createUser"
            >Create</div>
            <div
                v-if="selected === 'logout'"
                class="loginSubmit"
                v-on:click="logout"
            >Logout</div>
            <div
                class="xhrstatus"
            >{{xhrstatus}}</div>
        </div>
    </div>
</template>

<script lang="ts">
    import '../assets/colorVars.css';
    import {config} from "../assets/config";
    import xhr from "xhr";
    import LoginHeader from "./LoginPageComponents/LoginHeader.vue";
    import Vue from "vue";

    export default Vue.extend({
        name: "LoginPage",
        components: {
            LoginHeader
        },
        data () {
            return {
                username: "",
                usernum: "",
                password: "",
                xhrstatus: "",
                selected: "login",
                selectedNum: 0,
            }
        },
        methods: {
            changeSelected(arg, arg2) {
                this.selected = arg;
                this.selectedNum = arg2;
            },
            loginUser () {
                const self = this;
                xhr({
                    method: "POST",
                    body: JSON.stringify({
                        username: this.username,
                        usernum: this.usernum,
                        password: this.password
                    }),
                    uri: config.serverUrl + "/login",
                    headers: {
                        "Content-Type": "application/json",
                        "Access-Control-Allow-Origin": "<origin> | *"
                    }
                }, (err, resp, body) => {
                    if (err) throw err;
                    body = JSON.parse(body);
                    if (body.status === "success") {
                        localStorage.setItem("username", self.username);
                        localStorage.setItem("password", self.password);
                        localStorage.setItem("usernum", body.usernum);
                        localStorage.setItem("seshkey", body.seshkey);
                        this.$router.replace("/app");
                    } else {
                        self.xhrstatus = body.desc;
                    }
                })
            },
            createUser () {
                const self = this;
                xhr({
                    method: "POST",
                    body: JSON.stringify({
                        username: this.username,
                        password: this.password
                    }),
                    uri: config.serverUrl + "/createUser",
                    useXDR: true,
                    headers: {
                        "Content-Type": "application/json",
                        "Access-Control-Allow-Origin": "*",
                    }
                }, (err, resp, body) => {
                    if (err) throw err;
                    if (resp.statusCode !== 200) {
                        console.log(resp.statusCode);
                    }
                    self.xhrstatus = body;
                    if (JSON.parse(body).status === "Success") {
                        self.usernum = JSON.parse(body).usernum;
                        localStorage.setItem("newUser", "true");
                        this.loginUser();
                    }
                });
            },
            logout () {
                const self = this;
                xhr({
                    method: "POST",
                    body: JSON.stringify({
                        seshkey: localStorage.getItem("seshkey"),
                    }),
                    uri: config.serverUrl + "/logout",
                    useXDR: true,
                    headers: {
                        "Content-Type": "application/json",
                        "Access-Control-Allow-Origin": "*",
                    }
                }, (err, resp) => {
                    if (err) throw err;
                    if (resp.statusCode !== 200) {
                        console.log(resp.statusCode);
                    }
                    localStorage.removeItem("seshkey");
                    self.xhrstatus = "Logout successful.";
                });
            }
        }
    })
</script>

<style scoped lang="scss">
    @import url('https://fonts.googleapis.com/css?family=Montserrat&display=swap');
    .loginPage {
        background-image: linear-gradient(160deg, var(--aquifer-medium-3) 49%, var(--aquifer-medium-2) 51%);
        min-height: 100vh;
        height: 100vh;
    }
    .loginBox {
        width: 50%;
        height: 45%;
        padding: 5%;
        border-radius: 10px;
        position: absolute;
        left: 20%;
        top: 22.5%;
        background-color: var(--aquifer-dark-3);
        color: azure;
        text-align: left;
        font-family: Montserrat, 'Open Sans',  sans-serif;
    }
    .loginHeaders {
        display: flex;
        flex-wrap: nowrap;
        justify-content: center;
    }
    .loginInput {
        margin: 5% 0 0 5%;
        width: calc(82%);
        height: 30px;
        padding: 10px;
        font-size: 24px;
        background-color: var(--aquifer-light-1);
        border: 3px var(--aquifer-light-2) solid;
        border-radius: 5px;
        color: var(--aquifer-text-dark-2);
    }
    .loginUsername {
        margin: 5% 5% 0 0;
        height: 30px;
        padding: 2%;
        font-size: 24px;
        background-color: var(--aquifer-light-1);
        border: 3px var(--aquifer-light-2) solid;
        border-radius: 5px;
        color: var(--aquifer-text-dark-2);
    }
    .loginUsernum {
        margin-top: 5%;
        width: 20%;
        height: 30px;
        padding: 2%;
        font-size: 24px;
        background-color: var(--aquifer-light-1);
        border: 3px var(--aquifer-light-2) solid;
        border-radius: 5px;
        color: var(--aquifer-text-dark-2);
    }
    .usernameInputs {
        margin: auto;
        width: 90%;
    }
    .loginSubmit {
        width: 85%;
        margin: 5%;
        height: 40px;
        line-height: 40px;
        border: 3px var(--aquifer-light-1) solid;
        text-align: center;
        display: inline-block;
        background-color: var(--aquifer-medium-4);
        transition: background-color 0.2s;
    }
    .loginSubmit:hover {
        background-color: var(--aquifer-medium-1);
    }
    .xhrstatus {
        text-align: center;
    }
    .welcomeMsg {
        font-family: Montserrat, Calibri, sans-serif;
        font-size: 10vh;
        color: var(--aquifer-light-2);
        margin: 0;
        padding-top: 5vh;
        user-select: none;
    }
</style>
