<template>
    <div class="page__center wide">
        <div class="page__row page__row_head" v-if="!mobile">
            <div class="page__col col__header d-flex flex-column flex-sm-row justify-content-space-between">
                <div class="mt-4 mt-sm-0">
                    <div class="page__hello h5" v-if="user" v-html="user.name+','"/>
                    <div class="page__welcome h2">Привет 👋</div>
                </div>

                <div class="mt-4 mt-sm-0">
                    <p class="mb-2">
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <circle class="color" cx="7" cy="7" r="6.5" fill="#7FBA7A"/>
                        </svg>

                        <span class="color-gray ml-2 align-middle">На продвижении</span>
                    </p>
                    <p>
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <circle class="color" cx="7" cy="7" r="6.5" fill="#FF4242"/>
                        </svg>

                        <span class="color-gray ml-2 align-middle">Есть заявки</span>
                    </p>
                </div>
            </div>
        </div>

        <div class="page__row" v-if="mobile">
            <div class="d-flex flex-row align-items-center justify-content-between">
                <h1>Мои треки</h1>
                <img :src="url + 'img/icon_plus_main.svg'" @click="openAddMusicModal" class="mb-2" />
            </div>
        </div>

        <div class="page__row" v-if="mobile">
             <div class="page__panel">
                <p>
                    <svg width="12" height="12" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <circle class="color" cx="6" cy="6" r="5.5" fill="#7FBA7A"/>
                    </svg>

                    <span>На продвижении</span>
                </p>
                <p>
                    <svg width="12" height="12" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <circle class="color" cx="6" cy="6" r="5.5" fill="#FF4242"/>
                    </svg>

                    <span>Есть заявки</span>
                </p>
            </div>
        </div>

        <div class="page__row page__row_border">
            <div class="page__col">

                <div class="products__banner" v-if="items.length <= 0">
                    <div class="products__title title">Загрузи свой первый трек</div>
                    <p class="fan_text">Твои будущие фанаты ждут! Жми на кнопку «добавить трек» и переходи к продвижению прямо сейчас.</p>
                    <div class="btn btn-primary banner-btn" @click="openAddMusicModal">
                      <div><img src="/images/icon_plus_primary.svg"></img></div>
                      <span>Добавить трек</span>
                    </div>
                    <div class="guitar-image"></div>
                </div>

                <div class="products__grid" v-else>

                    <div class="products__item" @click="openAddMusicModal" v-if="!mobile">
                        <div class="products__preview new"></div>
                        <div class="products__details">
                            <div class="products__title title">Добавить трек</div>
                        </div>
                    </div>

                    <div class="products__item" v-for="item in items">
                        <div class="products__preview"><img v-bind:src="item.image"></div>
                        <div class="products__details">
                            <div class="products__title title" v-html="item.title"></div>
                        </div>
                    </div>

                </div>

            </div>
        </div>

        <b-modal id="add-music-modal" class="add-music-modal" centered hide-footer>
            <div class="modal-center d-flex flex-column text-center mx-auto">
                <div class="form-block">
                    <input type="text" class="form-control" placeholder="Ссылка на звук в TikTok" v-model="val" required="" />
                    <p class="form-tip text-danger" v-if="error" v-html="error" />
                    <button class="btn btn-lg btn-primary btn-block my-4" @click="getMusic(val)" :disabled="!val" v-if="!waiting" v-html="val ? 'Найти трек' : 'Введите ссылку на трек'" />
                    <div class="loading" :class="{active: waiting}" />
                    <p class="form-tip" v-if="waiting" v-html="'Ищем трек, это займет от 5 до 10 секунд'" />
                </div>
            </div>
        </b-modal>

        <b-modal id="edit-music-modal" centered hide-footer>
            <div class="avatar" v-bind:src="image"></div>
            <div class="form-block-new">
              <div class="form-block__text">
                <p class="label">Ссылка на звук в TikTok</p>
                <input id="form_1" type="text" class="form-control" placeholder="Ссылка на звук в TikTok" v-model="link" required="" />
              </div>

              <div class="form-block__text">
                <p class="label">Название</p>
                <input id="form_2" type="text" class="form-control" placeholder="Название трека" v-model="title" required="" />
              </div>

              <div class="form-block__text">
                <p class="label">Исполнитель</p>
                <input id="form_3" type="text" class="form-control" placeholder="Исполнитель трека" v-model="author" required="" />
              </div>

              <div class="form-block__text">
                <p class="label">Альбом</p>
                <input id="form_4" type="text" class="form-control" placeholder="Альбом трека" v-model="album" required="" />
              </div>

              <p class="form-tip text-danger" v-if="error" v-html="error" />
              <div class="link-button">
                <button class="btn btn-lg btn-primary btn-block my-4" @click="addMusic(link, title, author, album)" :disabled="!link" v-if="!waiting" v-html="link  ? 'Добавить' : 'Заполните формы'" />
                <div class="loading" :class="{active: waiting}" />
                <p class="form-tip" v-if="waiting" v-html="'Ищем трек, это займет от 5 до 10 секунд'" />
              </div>
            </div>
        </b-modal>


    </div>
</template>

<script>
    import axios from "axios"
    import {mapGetters} from "vuex";
    import { GET_MUSIC_LIST, GET_MUSIC, ADD_MUSIC } from '../api-routes';

    export default {
        components: { },
        name: "Music",

        data() {
            return {
                editing: false,
                val: this.value,
                image: this.value,
                link: this.value,
                title: this.value,
                author: this.value,
                album: this.value,
                music: null,
                error: null,
                waiting: false,
                items: [],
            }
        },

        mounted() {
            this.getMusicList();
        },

        computed: {
            ...mapGetters(['user', 'userAuthorized', 'url', 'tablet', 'mobile']),
        },

        methods: {
            openAddMusicModal() {
                this.waiting = this.error = false;
                this.$bvModal.show('add-music-modal');
            },
            openEditMusicModal() {
                this.waiting = this.error = false;
                this.$bvModal.show('add-music-modal');
            },

            getMusicList() {
                axios.get(GET_MUSIC_LIST).then(response => {
                    this.items = response.data;
                });
            },

            addMusic(url, title, author, album) {
                this.waiting = true;
                this.error = null;

                axios.post(ADD_MUSIC, {url: url, title: title, author: author, album: album}).then(response => {
                    this.waiting = false;

                    /* TO DO */
                    window.location.reload();
                    console.log(response);

                    this.error = null;
                }).catch(error => {
                    console.log(error);
                    this.waiting = false;
                    if(error !== undefined) {
                        this.error = 'Не удалось добавить трек, попробуйте еще раз';
                    }
                });
            },

            getMusic(url) {
                this.waiting = true;
                this.error = null;
                axios.post(GET_MUSIC, {url: url}).then(response => {
                    this.waiting = false;
                    /* TO DO */
                    this.link = this.val;
                    console.log(url);
                    this.title = response.data.music.title;
                    this.author = response.data.music.authorName;
                    this.album = response.data.music.album;
                    this.image = response.data.music.coverThumb;
                    this.$bvModal.show('edit-music-modal');
                    console.log(response);

                    this.error = null;
                }).catch(error => {
                    console.log(error);
                    this.waiting = false;
                    if(error !== undefined) {
                        this.error = 'Не удалось найти трек, попробуйте еще раз';
                    }
                });
            }
        }

    }
</script>
