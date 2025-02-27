<script setup lang="ts">
  useHead({
    title: "Homebox | Organize and Tag Your Stuff",
  });

  definePageMeta({
    layout: "empty",
  });

  const ctx = useAuthContext();
  if (ctx.isAuthorized()) {
    navigateTo("/home");
  }

  const api = usePublicApi();
  const toast = useNotifier();

  const { data: status } = useAsyncData(async () => {
    const { data } = await api.status();

    if (data.demo) {
      username.value = "demo@example.com";
      password.value = "demo";
    }
    return data;
  });

  whenever(status, status => {
    if (status?.demo) {
      email.value = "demo@example.com";
      loginPassword.value = "demo";
    }
  });

  const route = useRoute();
  const router = useRouter();

  const username = ref("");
  const email = ref("");
  const password = ref("");
  const canRegister = ref(false);
  const remember = ref(false);

  const groupToken = computed<string>({
    get() {
      const params = route.query.token;

      if (typeof params === "string") {
        return params;
      }

      return "";
    },
    set(v) {
      router.push({
        query: {
          token: v,
        },
      });
    },
  });

  async function registerUser() {
    loading.value = true;
    const { error } = await api.register({
      name: username.value,
      email: email.value,
      password: password.value,
      token: groupToken.value,
    });

    if (error) {
      toast.error("Problem registering user");
      return;
    }

    toast.success("User registered");

    loading.value = false;
    registerForm.value = false;
  }

  onMounted(() => {
    if (groupToken.value !== "") {
      registerForm.value = true;
    }
  });

  const loading = ref(false);
  const loginPassword = ref("");

  async function login() {
    loading.value = true;
    const { error } = await ctx.login(api, email.value, loginPassword.value, remember.value);

    if (error) {
      toast.error("Invalid email or password");
      loading.value = false;
      return;
    }

    toast.success("Logged in successfully");

    navigateTo("/home");
    loading.value = false;
  }

  const [registerForm, toggleLogin] = useToggle();
</script>

<template>
  <div class="flex flex-col min-h-screen">
    <div class="fill-primary min-w-full absolute top-0 z-[-1]">
      <div class="bg-primary flex-col flex min-h-[20vh]" />
      <svg
        class="fill-primary drop-shadow-xl"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 1440 320"
        preserveAspectRatio="none"
      >
        <path
          fill-opacity="1"
          d="M0,32L80,69.3C160,107,320,181,480,181.3C640,181,800,107,960,117.3C1120,128,1280,224,1360,272L1440,320L1440,0L1360,0C1280,0,1120,0,960,0C800,0,640,0,480,0C320,0,160,0,80,0L0,0Z"
        ></path>
      </svg>
    </div>
    <div>
      <header class="p-4 sm:px-6 lg:p-14 sm:py-6 sm:flex sm:items-end mx-auto">
        <div>
          <h2 class="mt-1 text-4xl font-bold tracking-tight text-neutral-content sm:text-5xl lg:text-6xl flex">
            HomeB
            <AppLogo class="w-12 -mb-4" />
            x
          </h2>
          <p class="ml-1 text-lg text-base-content/50">Track, Organize, and Manage your Things.</p>
        </div>
        <div class="flex mt-6 sm:mt-0 gap-4 ml-auto text-neutral-content">
          <a class="tooltip" data-tip="Project Github" href="https://github.com/hay-kot/homebox" target="_blank">
            <Icon name="mdi-github" class="h-8 w-8" />
          </a>
          <a href="https://twitter.com/haybytes" class="tooltip" data-tip="Follow The Developer" target="_blank">
            <Icon name="mdi-twitter" class="h-8 w-8" />
          </a>
          <a href="https://discord.gg/tuncmNrE4z" class="tooltip" data-tip="Join The Discord" target="_blank">
            <Icon name="mdi-discord" class="h-8 w-8" />
          </a>
          <a href="https://hay-kot.github.io/homebox/" class="tooltip" data-tip="Read The Docs" target="_blank">
            <Icon name="mdi-folder" class="h-8 w-8" />
          </a>
        </div>
      </header>
      <div class="grid p-6 sm:place-items-center min-h-[50vh]">
        <div>
          <Transition name="slide-fade">
            <form v-if="registerForm" @submit.prevent="registerUser">
              <div class="card w-max-[500px] md:w-[500px] bg-base-100 shadow-xl">
                <div class="card-body">
                  <h2 class="card-title text-2xl align-center">
                    <Icon name="heroicons-user" class="mr-1 w-7 h-7" />
                    Register
                  </h2>
                  <FormTextField v-model="email" label="Set your email?" />
                  <FormTextField v-model="username" label="What's your name?" />
                  <div v-if="!(groupToken == '')" class="pt-4 pb-1 text-center">
                    <p>You're Joining an Existing Group!</p>
                    <button type="button" class="text-xs underline" @click="groupToken = ''">
                      Don't Want To Join a Group?
                    </button>
                  </div>
                  <FormPassword v-model="password" label="Set your password" />
                  <PasswordScore v-model:valid="canRegister" :password="password" />
                  <div class="card-actions justify-end">
                    <button
                      type="submit"
                      class="btn btn-primary mt-2"
                      :class="loading ? 'loading' : ''"
                      :disabled="loading || !canRegister"
                    >
                      Register
                    </button>
                  </div>
                </div>
              </div>
            </form>
            <form v-else @submit.prevent="login">
              <div class="card w-max-[500px] md:w-[500px] bg-base-100 shadow-xl">
                <div class="card-body">
                  <h2 class="card-title text-2xl align-center">
                    <Icon name="heroicons-user" class="mr-1 w-7 h-7" />
                    Login
                  </h2>
                  <template v-if="status && status.demo">
                    <p class="text-xs italic text-center">This is a demo instance</p>
                    <p class="text-xs text-center"><b>Email</b> demo@example.com</p>
                    <p class="text-xs text-center"><b>Password</b> demo</p>
                  </template>
                  <FormTextField v-model="email" label="Email" />
                  <FormPassword v-model="loginPassword" label="Password" />
                  <div class="max-w-[140px]">
                    <FormCheckbox v-model="remember" label="Remember Me" />
                  </div>
                  <div class="card-actions justify-end">
                    <button
                      type="submit"
                      class="btn btn-primary btn-block"
                      :class="loading ? 'loading' : ''"
                      :disabled="loading"
                    >
                      Login
                    </button>
                  </div>
                </div>
              </div>
            </form>
          </Transition>
          <div class="text-center mt-6">
            <BaseButton
              v-if="status && status.allowRegistration"
              class="btn-primary btn-wide"
              @click="() => toggleLogin()"
            >
              <template #icon>
                <Icon v-if="!registerForm" name="mdi-account-plus-outline" class="w-5 h-5 swap-off" />
                <Icon v-else name="mdi-login" class="w-5 h-5 swap-off" />
                <Icon name="mdi-arrow-right" class="w-5 h-5 swap-on" />
              </template>
              {{ registerForm ? "Login" : "Register" }}
            </BaseButton>
            <p v-else class="text-base-content italic text-sm inline-flex items-center gap-2">
              <Icon name="mdi-lock" class="w-4 h-4 inline-block" />
              Registration Disabled
            </p>
          </div>
        </div>
      </div>
    </div>
    <footer v-if="status" class="mt-auto text-center w-full bottom-0 pb-4">
      <p class="text-center text-sm">Version: {{ status.build.version }} ~ Build: {{ status.build.commit }}</p>
    </footer>
  </div>
</template>

<style lang="css" scoped>
  .slide-fade-enter-active {
    transition: all 0.2s ease-out;
  }

  .slide-fade-enter-from,
  .slide-fade-leave-to {
    position: absolute;
    transform: translateX(20px);
    opacity: 0;
  }

  progress[value]::-webkit-progress-value {
    transition: width 0.5s;
  }
</style>
