<script>
import { db } from "../firebase.ts"
import { query, collection, getDocs, orderBy, updateDoc, doc, setDoc,deleteDoc   } from "firebase/firestore";
import { useStundetStore } from '../stores/students'


// await setDoc(doc(db, "drops", `${item.id}`), item);

export default {
    data() {
        return {
            store: useStundetStore(), // от пиниа
            login: '', // логин админа
            password: '', // пароль админа
            passed: true, // прошёл ли админ авторизацию
            error: false, // ошибка при авторизации

            arr: [], // все месяца
            eventID: '', // какой сейчас выбран месяц
            newMonth: '', // инпут при добавлении месяца (model)
            showMonth: [], // показ месяцев

            addFallback: [], // ПОДГОТОВКА С СОХРАНЕНИЮ
            addName: '', // Добавление в fallback
            addId: '', // Добавление в fallback
            addEnglishTeacher: '', // Добавление в fallback
            addEnglishMark: '', // Добавление в fallback
            addMathTeacher: '', // Добавление в fallback
            addMathMark: '', // Добавление в fallback
            addItTeacher: '', // Добавление в fallback
            addItMark: '', // Добавление в fallback
        }
    },
    created() {
        this.getItems()
        
    },
    methods: {
        // получить массив из firebase
        async getItems() {
            const q = query(collection(db, "exam"))
            const querySnap = await getDocs(q);
            querySnap.forEach((doc) => {
                this.arr.push(doc.data())
            })


        },
        // проверка пароля и логина
        check() {
            this.login == "mr.admin" && this.password == "9248077Asd" ? this.sign() : this.errors()
        },
        // вход
        sign() {
            this.passed = true
        },
        // ошибка при входе
        errors() {
            this.passed = false
            this.error = true
        },
        // функция при клике на месяца
        show(item) {
            this.eventID = event.target.value
            this.showMonth = this.arr[this.eventID]
            let items = document.querySelectorAll(".categories")
            // for (let i of items) i.value == this.eventID ? i.classList.add("active") : i.classList.remove("active")
            for (let i of items) {
                if (i.value == this.eventID) {
                    i.classList.add("active")
                    i.nextElementSibling.classList.add("asideActive")
                } else {
                    i.classList.remove("active")
                    i.nextElementSibling.classList.remove("asideActive")
                }
            }
        },
        // добавление экзамена 
        async addExam() {
            let m = Math.random().toString().substr(2, 10)
            await setDoc(doc(db, "exam", m), {
                info: {
                    title: this.newMonth,
                    idx: m,
                },
                students: []
            });
            this.arr = []
            this.getItems()
        },
        // обговить 
        reloadFunc() {
            this.arr = []
            this.showMonth = []
            this.getItems()
        },
        // добавление в фолбек
        addItemToFallback() {
            let student = {
                name: this.addName,
                id: this.addId,
                subjects: {
                    english: {
                        mark: this.addEnglishMark,
                        teacher: this.addEnglishTeacher,
                    },
                    math: {
                        mark: this.addMathMark,
                        teacher: this.addMathTeacher,
                    },
                    it: {
                        mark: this.addItMark,
                        teacher: this.addItTeacher,
                    },
                }
            }
            this.addFallback.push(student)
        },
        // удалить из фолбека 
        deleteInFallback(item) {
            let filtered = this.addFallback.filter(students => students.id != item.id)
            this.addFallback = filtered
        },

        // сохранить добавление в фолбеке 
        async saveFallback() {
            let concats = this.arr[this.eventID]
            for (let i of this.addFallback) {
                concats.students.push(i)
            }

            await setDoc(doc(db, "exam", this.arr[this.eventID].info.idx), concats);
        },

        // // debug 
        // actionss(){
        //     this.addFallback = [] надо поставить объекты 
        //     this.saveFallback()   и кнш пушить

        // },


        // удаление из firebase 
        async deleteItem(items) {
            let filtered = this.arr[this.eventID].students.filter(item => item.id !== items.id)
            this.arr[this.eventID].students = filtered
            await setDoc(doc(db, "exam", this.arr[this.eventID].info.idx), this.arr[this.eventID]);
        },
        // удаление месяца 
        async deleteMonth(items) {
           
            // await setDoc(doc(db, "exam", 0), this.arr);
            // let filtered = this.arr[this.eventID].students.filter(item => item.id !== items.id)
            // this.arr[this.eventID].students = filtered
            await deleteDoc(doc(db, "exam", this.arr[this.eventID].info.idx) )
            let filtered = this.arr.filter(item => item != items)
            this.arr = filtered

        }



    }
}

</script>
<template>
    <!-- <button @click="actionss">click</button> for adding  -->
    <main>
        <!-- вход  -->
        <div v-if="!passed" class="flex min-h-full flex-col justify-center px-6 py-12 lg:px-8">
            <div class="sm:mx-auto sm:w-full sm:max-w-sm">
                <img class="mx-auto h-[100px] w-auto" src="https://api.modme.uz/uploads/cTnyBMNN7kXaQlav.png"
                    alt="Your Company">
                <h2 class="mt-10 text-center text-2xl font-bold leading-9 tracking-tight text-gray-900">Войти в админ
                    панель</h2>
                <h2 v-if="this.error" class="text-center text-[15px] font-bold leading-9 tracking-tight text-[red]">
                    Пароль или логин были введены неправильно</h2>
            </div>

            <div class="mt-10 sm:mx-auto sm:w-full sm:max-w-sm">
                <form @submit.prevent="check()" class="space-y-6" action="#" method="POST">
                    <div>
                        <label for="email" class="block text-sm font-medium leading-6 text-gray-900">Логин:</label>
                        <div class="mt-2">
                            <input v-model="login" id="email" name="email" type="text" autocomplete="email" required
                                class="pl-[10px] outline-none block w-full rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6">
                        </div>
                    </div>

                    <div>
                        <div class="flex items-center justify-between">
                            <label for="password"
                                class="block text-sm font-medium leading-6 text-gray-900">Пароль:</label>
                        </div>
                        <div class="mt-2">
                            <input v-model="password" id="password" name="password" type="password"
                                autocomplete="current-password" required
                                class="block outline-none pl-[10px] w-full rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6">
                        </div>
                    </div>

                    <div>
                        <button type="submit"
                            class="flex w-full justify-center rounded-md bg-[#FF1616] px-3 py-1.5 text-sm font-semibold leading-6 text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">Войти</button>
                    </div>
                </form>

                <p class="mt-10 text-center text-sm text-gray-500">
                    Если вы попали сюда случайно то
                    <a href="https://miamiexam.org/"
                        class="font-semibold leading-6 text-indigo-600 hover:text-indigo-500">нажмите сюда</a>
                </p>
            </div>
        </div>
        <div v-else class="min-h-full">
            <header class="bg-white shadow">
                <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
                    <h1 class="text-3xl font-bold tracking-tight text-gray-900">Админ панель</h1>
                </div>
            </header>
            <main>
                <div class="mx-auto flex max-w-7xl py-6 sm:px-6 lg:px-8 m450">
                    <div
                        class="max-w-[20%] w-[100%] m451 min-h-[400px] pr-[20px] h-[max-content] border-r-[1px] border-[red] mr-[20px]">
                        <ul>
                            <li class="mb-[10px] font-[400]">
                                Проведённые экзамены:
                            </li>
                            <li class="relative" v-for="item of this.arr" :key="item.id">
                                <button @click="show()" :value="this.arr.indexOf(item)"
                                    class="categories btn outline-none mb-[10px] w-[100%]">

                                    {{ item.info.title
                                    }}
                                </button>
                                <aside class="aside" @click="deleteMonth(item)">
                                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                                        stroke-width="1.5" stroke="currentColor" class="w-5 h-5">
                                        <path stroke-linecap="round" stroke-linejoin="round"
                                            d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0" />
                                    </svg>
                                </aside>
                            </li>
                            <li>
                                <button onclick="my_modal_4.showModal()" :value="'item'"
                                    class="btn outline-none mb-[10px] hover:bg-[#ff0000b9] w-[100%] bg-[red] text-[white]">Добавить</button>
                                <dialog id="my_modal_4" class="modal">
                                    <div class="modal-box w-11/12 max-w-4xl">
                                        <h3 class="font-bold text-lg">Добавить экзамен</h3>
                                        <p class="py-4">Месяц и год (на английском)</p>
                                        <input v-model="newMonth"
                                            class="border-[red] outline-none text-[17px] font-[600] border-[2px] px-[20px] py-[10px] w-[100%] rounded-[8px]"
                                            placeholder="Прмер: September 2023" type="text">
                                        <div class="modal-action w-[100%]">
                                            <form class="w-[100%]" method="dialog">
                                                <!-- if there is a button, it will close the modal -->
                                                <div class="flex gap-[10px] w-[100%]">
                                                    <button class="btn w-[50%]">Close</button>
                                                    <button @click="addExam()"
                                                        class="btn bg-[red] text-white hover:bg-[#ff0000c4] w-[50%]">Добавить</button>
                                                </div>
                                            </form>
                                        </div>
                                    </div>
                                </dialog>
                            </li>
                        </ul>
                    </div>
                    <div class="flex-1 m451 min-h-[80vh] h-[max-content] m450cont">
                        <div v-if="this.showMonth.length < 1">
                            <h1>Сначала выберите месяц чтобы увидеть подробную информацию.</h1>
                        </div>
                        <div v-else>
                            <div class="flex justify-between items-center m450title">
                                <h1 class="font-[700] text-[25px] capitalize"> {{ this.showMonth.info.title }} </h1>
                                <div class="flex items-center">
                                    <button @click="reloadFunc()" class="btn bg-[red] text-white">
                                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                                            stroke-width="1.5" stroke="currentColor" class="w-5 h-5">
                                            <path stroke-linecap="round" stroke-linejoin="round"
                                                d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                                        </svg>
                                    </button>
                                    <button onclick="my_modal_10.showModal()" class="btn bg-[red] text-white">Добавить
                                        ученика</button>
                                    <dialog id="my_modal_10" class="modal">
                                        <div class="modal-box w-[100%] max-w-[90%]">
                                            <h3 class="font-bold text-lg">Добавить ученика в "{{
            this.arr[this.eventID].info.title }}"</h3>
                                            <div class="modal-action">
                                                <form class="w-[100%]" method="dialog">
                                                    <div class="min-h-[100px] m450pop mb-[10px] gap-[10px] flex">
                                                        <div class="flex m450pop1 flex-col flex-1">
                                                            <span>Имя</span>
                                                            <input v-model="addName"
                                                                class="px-[10px] border py-[8px] rounded-[6px]"
                                                                placeholder="Иван Иванов" type="text">
                                                        </div>
                                                        <div class="flex flex-col flex-1">
                                                            <span>ID</span>
                                                            <input v-model="addId"
                                                                class="px-[10px] border py-[8px] rounded-[6px]"
                                                                placeholder="AH2765" type="text">
                                                        </div>
                                                        <div class="flex flex-col flex-1">
                                                            <span>English</span>
                                                            <input v-model="addEnglishTeacher" clas
                                                                placeholder="Учитель" type="text"
                                                                class="mb-[5px] px-[10px] border py-[8px] rounded-[6px]">
                                                            <input v-model="addEnglishMark"
                                                                class="px-[10px] border py-[8px] rounded-[6px]"
                                                                placeholder="Оценка: 74/100" type="text">
                                                        </div>
                                                        <div class="flex flex-col flex-1">
                                                            <span>Math</span>
                                                            <input v-model="addMathTeacher" placeholder="Учитель"
                                                                type="text"
                                                                class="mb-[5px] px-[10px] border py-[8px] rounded-[6px]">
                                                            <input v-model="addMathMark"
                                                                class="px-[10px] border py-[8px] rounded-[6px]"
                                                                placeholder="Оценка: 74/100" type="text">
                                                        </div>
                                                        <div class="flex flex-col flex-1">
                                                            <span>IT</span>
                                                            <input v-model="addItTeacher" placeholder="Учитель"
                                                                type="text"
                                                                class="mb-[5px] px-[10px] border py-[8px] rounded-[6px]">
                                                            <input v-model="addItMark"
                                                                class="px-[10px] border py-[8px] rounded-[6px]"
                                                                placeholder="Оценка: 74/100" type="text">
                                                        </div>
                                                        <div class="flex flex-col flex-1">
                                                            <div @click="addItemToFallback"
                                                                class="bg-[red] text-center cursor-pointer mt-[25px] text-white py-[8px] rounded-[8px]">
                                                                Далее</div>
                                                        </div>
                                                    </div>
                                                    <div
                                                        class="min-h-[100px] border-t-[1px] border-[red] mt-[50px] mb-[20px]">
                                                        <div class="overflow-x-auto"
                                                            style="max-height: 400px; overflow-y: scroll;">
                                                            <table class="table">
                                                                <!-- head -->
                                                                <thead>
                                                                    <tr>
                                                                        <th>Имя</th>
                                                                        <th class="w-[100px]">ID</th>
                                                                        <th class="w-[200px]">English</th>
                                                                        <th class="w-[200px]">Math</th>
                                                                        <th class="w-[200px]">IT</th>
                                                                        <th class="w-[100px]">Action</th>
                                                                    </tr>
                                                                </thead>
                                                                <tbody>
                                                                    <!-- row 1 -->
                                                                    <tr v-for="fall of this.addFallback" :key="fall.id">
                                                                        <td>{{ fall.name }}</td>
                                                                        <td class="w-[max-content]">{{ fall.id }}</td>
                                                                        <td>
                                                                            <div v-if="fall.subjects.english.mark.length > 0 && fall.subjects.english.teacher.length > 0"
                                                                                class="flex flex-col">
                                                                                <span class="text-[red] font-[500]">{{
            fall.subjects.english.mark }} - {{
            Math.round(
                (+fall.subjects.english.mark.split('/')[0]
                    * 100) /
                +fall.subjects.english.mark.split('/')[1]
            ) }}%</span>
                                                                                <span>{{ fall.subjects.english.teacher
                                                                                    }}</span>
                                                                            </div>
                                                                        </td>
                                                                        <td>
                                                                            <div v-if="fall.subjects.math.mark.length > 0 && fall.subjects.math.teacher != ''"
                                                                                class="flex flex-col">
                                                                                <span class="text-[red] font-[500]">{{
            fall.subjects.math.mark }} - {{
            Math.round(
                (+fall.subjects.math.mark.split('/')[0]
                    * 100) /
                +fall.subjects.math.mark.split('/')[1])
        }}%</span>
                                                                                <span>{{ fall.subjects.math.teacher
                                                                                    }}</span>
                                                                            </div>
                                                                        </td>
                                                                        <td>
                                                                            <div v-if="fall.subjects.it.mark != '' && fall.subjects.it.teacher != ''"
                                                                                class="flex flex-col">
                                                                                <span class="text-[red] font-[500]">{{
            fall.subjects.it.mark }} - {{
            Math.round((+fall.subjects.it.mark.split('/')[0]
                * 100) /
                +fall.subjects.it.mark.split('/')[1])
        }}%</span>
                                                                                <span>{{ fall.subjects.it.teacher
                                                                                    }}</span>
                                                                            </div>
                                                                        </td>
                                                                        <td>
                                                                            <div @click="deleteInFallback(fall)"
                                                                                class="btn">
                                                                                <svg xmlns="http://www.w3.org/2000/svg"
                                                                                    fill="none" viewBox="0 0 24 24"
                                                                                    stroke-width="1.5"
                                                                                    stroke="currentColor"
                                                                                    class="w-6 h-6">
                                                                                    <path stroke-linecap="round"
                                                                                        stroke-linejoin="round"
                                                                                        d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0" />
                                                                                </svg>

                                                                            </div>
                                                                        </td>
                                                                    </tr>

                                                                </tbody>
                                                            </table>
                                                        </div>
                                                    </div>
                                                    <button class="btn mr-[20px]">Отмена</button>
                                                    <button @click="saveFallback"
                                                        class="btn bg-[red] text-white">Сохранить</button>
                                                </form>
                                            </div>
                                        </div>
                                    </dialog>
                                </div>
                            </div>
                            <div class="overflow-x-[auto] m450table">
                                <table class="table">
                                    <!-- head -->
                                    <thead>
                                        <tr>
                                            <th class="m450name">Имя</th>
                                            <th>ID</th>
                                            <th>English</th>
                                            <th>Math</th>
                                            <th>IT</th>
                                            <th>Action</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr v-if="this.showMonth.students.length < 1">
                                            <td>Тут пусто 🥵 Добавьте ученика.</td>
                                        </tr>
                                        <tr v-for="item of this.showMonth.students" :key="item.id">
                                            <td>{{ item.name }}</td>
                                            <td>{{ item.id }}</td>
                                            <th>
                                                <div
                                                    v-if="item.subjects.english.mark.length > 0 && item.subjects.english.teacher.length > 0">
                                                    <div>
                                                        <button class="text-[red]">{{ item.subjects.english.mark }}</button>
                                                    </div>
                                                    <span class="font-[400] text-[12px]">{{
                                                        item.subjects.english.teacher }}</span>

                                                </div>
                                            </th>
                                            <th>
                                                <div>   
                                                    <div>
                                                        <button class="text-[red]">{{ item.subjects.math.mark }}</button>
                                                    </div>
                                                    <span class="font-[400] text-[12px]">{{ item.subjects.math.teacher
                                                        }}</span>
                                                </div>
                                            </th>
                                            <th>
                                                <div
                                                    v-if="item.subjects.it.mark.length > 0 && item.subjects.it.teacher.length > 0">
                                                    <div>
                                                        <button class="text-[red]">{{ item.subjects.it.mark }} </button>
                                                    </div>
                                                    <span class="font-[400] text-[12px]">{{ item.subjects.it.teacher
                                                        }}</span>
                                                </div>
                                            </th>
                                            <th>
                                                <button @click="deleteItem(item)" class="btn text-[30px]">
                                                    <svg xmlns="http://www.w3.org/2000/svg" fill="none"
                                                        viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
                                                        class="w-5 h-5">
                                                        <path stroke-linecap="round" stroke-linejoin="round"
                                                            d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0" />
                                                    </svg>

                                                </button>
                                            </th>
                                        </tr>

                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                </div>
            </main>
        </div>
    </main>

</template>


<style>
.active {
    background: red;
    color: white;
    position: relative;
}

.aside {
    cursor: pointer;
    opacity: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    top: 0px;
    left: -60px;
    position: absolute;
    height: 40px;
    width: 40px;
    background: rgba(255, 0, 0, 0.742);
    border-radius: 10px;
}
.aside svg{
    color: white;
}

.asideActive{
    opacity: 1;
}

.active:hover {
    background: rgba(255, 0, 0, 0.742);
}

@media screen and (max-width: 450px) {
    .m450 {
        flex-direction: column;
        justify-content: flex-start;
        align-items: flex-start;
    }

    .m450 div:nth-child(1) {
        margin: 0px !important;
        border: none !important;
        min-height: max-content;
        min-width: initial !important;
        max-width: initial !important;
        width: 100%;
    }

    .m451 {
        padding: 20px !important;
    }

    .m450table {
        padding: 0 !important;
        max-width: calc(100vw - 50px);
        overflow-x: scroll !important;
    }

    .m450table th:nth-child(1) {
        width: 200px;
        min-width: 200px;
        overflow-x: scroll !important;

    }


    .m450title {
        flex-direction: column;
        align-items: flex-start;
        justify-content: flex-start
    }

    .m450cont {
        padding: 20px !important;
    }

    .m450name {
        min-width: 200px
    }

    .m450pop {
        display: flex;
        flex-direction: column;
        padding: 10px !important;
    }

    .m450pop1 {
        flex: initial !important;
        padding: 0px !important;
        margin: 0 !important;
    }


}
</style>