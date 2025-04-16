```js
export default {
    data() {
        return {
            form: {
                fio: null,
                birthday: null,
                male: null,
                tel: null,
                email: null,
                password: null
            },
            errors: {}
        }
    },
    methods:{
        validateForm(){
            this.errors = {}
            let valid = true
            if (!/^[А-Яа-яЁёA-Za-z\s\-]{5,100}$/.test(this.form.fio)) {
                this.errors.fio = 'ФИО должно содержать только буквы, пробелы'
                valid = false
            }
            if (!/^[0-9A-Za-z_.+-]{1,64}@[0-9A-Za-z_-]{1,10}\.[0-9A-Za-z_-]{2,6}$/.test(this.form.email)) {
                this.errors.email = 'Некорректный email'
                valid = false
            }
            if (!/^(?:\+7|8)\d{10}$/.test(this.form.tel)) {
                this.errors.tel = 'Телефон должен начинаться с +7 или 8 и содержать ровно 11 цифр';
                valid = false;
            }
            if (!/^(?=.*[.!,?])[0-9A-Za-z\s.!,?-]{8,40}$/.test(this.form.password)) {
                this.errors.password = 'Пароль должен быть от 8 символов и содержать спец символ(!.,?-)'
                valid = false
            }
            if (!this.form.male){
                this.errors.male = 'Выберите пол'
                valid = false
            }
            if (!this.form.birthday){
                this.errors.birthday = 'Выберите дату рождения'
                valid = false
            }
            return valid
        },
        async handleSubmit(){
            if (!this.validateForm()) {
                console.log('Всё плохо')
                return
            }
            try{
                const response = await axios.post(import.meta.env.VITE_BACK_URL+'/register',{
                    "fio":this.form.fio,
                    "birthday":this.form.birthday,
                    "male":this.form.male,
                    "tel":this.form.tel,
                    "email":this.form.email,
                    "password":this.form.password,
                })
            }
            catch(error){
                const msg = error.response?.data || 'Ошибка входа';
                alert(msg);
            }
            console.log('Всё чётко')
        }
    }
}

 async mounted() {
        const cachedTours = sessionStorage.getItem('tours');
        if(cachedTours){
            this.tours = JSON.parse(cachedTours);
        }else {
            this.fetchTours();
        }
    }


// vite.config.js
export default defineConfig({
  plugins: [vue(),pug(),vueDevTools()],
  server: {
    proxy: {
      '/data': {
        target: 'http://127.0.0.1:8000/', 
        changeOrigin: true,
      }
  }
}
})

getImageSrc(image) {
	return `/data/${image}`;
	// return new URL(`../../assets/images/tours/${image}`, import.meta.url).href;
},


methods:{
    sortProducts(){
      const key = this.filter;
      const sortMethods = {
        id: (a,b) => a.id - b.id,
        price: (a,b) => a.price - b.price,
        quantity: (a,b) => a.quantity - b.quantity,
        name: (a,b) => a.name.localeCompare(b.name),
        category: (a,b) => a.category.localeCompare(b.category),
        created_at: (a,b) => new Date(a.created_at) - new Date(b.created_at),
        updated_at: (a,b) => new Date(a.updated_at) - new Date(b.updated_at)
      }
      this.products.sort(sortMethods[key])
    }
  },
  watch: {
    filter(){
      this.sortProducts();
    }
  }


scrollBehavior(to, from, savedPosition) {
    if (to.hash) {
      return new Promise((resolve) => {
        setTimeout(() => {
          resolve({
            el: to.hash,  
            behavior: 'smooth',
          });
        }, 100);
      });
    } else if (savedPosition) {
	  return savedPosition;
    } else {
      return { top: 0 }
    }
  }


 export default {
  data() {
    return {
      selectedDate: '',
      minDate: '',
      maxDate: '',
    };
  },
  mounted() {
    const today = new Date();
    const max = new Date();

    max.setDate(today.getDate() + 7); // максимум через 7 дней

    this.minDate = today.toISOString().split('T')[0];
    this.maxDate = max.toISOString().split('T')[0];
  },
};
```
