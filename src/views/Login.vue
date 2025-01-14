<template>
	<div class="login">
		<div class="ui middle aligned center aligned grid">
			<div class="column">
				<h2 class="ui image header">
					<div class="content">账号</div>
					<img src="../assets/image/logo-pure.png" class="image" />
					<div class="content">登录</div>
				</h2>
				<div class="ui large form">
					<div class="ui stacked segment">
						<div class="field" :class="[emailInputStatus]">
							<div class="ui left icon input">
								<i class="user icon"></i>
								<input
									type="text"
									name="email"
									placeholder="邮箱"
									v-model="email"
									@focusout="onFocusOut"
								/>
							</div>
						</div>
						<div class="field">
							<div class="ui left icon input">
								<i class="lock icon"></i>
								<input
									type="password"
									name="password"
									placeholder="密码"
									v-model="password"
									@keypress="onKeyPress"
								/>
							</div>
						</div>
						<button
							class="ui fluid large teal submit button"
							:disabled="!emailExist || isEmpty(password)"
							@click="onLogin"
						>
							登录
						</button>
					</div>
				</div>
				<div class="ui error message" v-show="errorMessage != ''">
					<i class="close icon"></i>
					<div class="header">{{ errorMessage }}</div>
				</div>
				<div class="ui message">
					新用户？
					<router-link to="/register">加入我们</router-link>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
.login {
	width: 100%;
	height: 100%;
	display: flex;
	justify-content: center;
	align-items: center;
	background-image: url("../assets/image/background/login.png");
	background-size: cover;
}
.login .ui.grid {
	min-width: 25em;
}
.login .ui.grid .column {
	opacity: 0.8;
}
.login .ui.grid .column .ui.header {
	display: flex;
	align-items: center;
	justify-content: center;
	margin-bottom: 1em;
}
.login .ui.grid .column .ui.header .content {
	padding: 0.5em;
	color: #223a55;
}
.login .ui.grid .column .ui.segment,
.login .ui.grid .column .ui.message {
	background-color: rgba(255, 255, 255, 0.6);
}
</style>

<script lang="ts">
import { Vue, Options } from "vue-class-component";
import { RouteLocationNormalized, RouteLocationRaw } from "vue-router";
import Axios, { AxiosError } from "axios";
import $ from "jquery";

type RouteLocationSimple = Pick<
	RouteLocationNormalized,
	"name" | "query" | "params"
>;
class Props {
	pFrom?: string;
}
@Options({
	emits: {
		toggleHeader: (visible: boolean) => typeof visible == "boolean",
		setBackgroundImage: (filename: string) => typeof filename == "string",
	},
	watch: {
		emailExist(this: LoginView, value) {
			if (value == false) this.errorMessage = "邮箱未注册";
			else this.errorMessage = "";
		},
	},
})
export default class LoginView extends Vue.with(Props) {
	from?: RouteLocationSimple;
	email: string = "";
	password: string = "";
	emailExist: boolean | null = null;
	errorMessage: string = "";

	get emailInputStatus() {
		switch (this.emailExist) {
			case true:
				return "success";
			case false:
				return "error";
			default:
				return "";
		}
	}
	get loginError() {
		return this.errorMessage != "";
	}

	isEmpty(str: string): boolean {
		return str == "";
	}
	onFocusOut() {
		if (this.isEmpty(this.email)) this.emailExist = null;
		else if (!/^\w+@\w+\.+\w+$/.test(this.email)) this.emailExist = false;
		else {
			Axios.get("/api/user/email", {
				params: {
					email: this.email,
				},
			}).then(response => {
				this.emailExist = (response.data as any).exist;
			});
		}
	}
	onLogin() {
		const target: RouteLocationRaw = {};
		if (this.from) Object.assign(target, this.from);
		else target.name = "Home";
		Axios.post("/api/user/login", {
			email: this.email,
			password: this.password,
		}).then(
			_ => this.$router.push(target),
			(error: AxiosError) => {
				if (error.response!.status == 400) this.$router.push(target);
				else this.errorMessage = error.response?.data as string;
			}
		);
	}
	onKeyPress(key: KeyboardEvent) {
		if (key.code == "Enter") this.onLogin();
	}

	created() {
		if (this.pFrom) this.from = JSON.parse(this.pFrom);
	}
	mounted() {
		this.$emit("toggleHeader", false);
		$(".icon.close", this.$el).on("click", function() {
			$(this)
				.parent()
				.hide();
		});
	}
	unmounted() {
		this.$emit("toggleHeader", true);
	}
}
</script>
