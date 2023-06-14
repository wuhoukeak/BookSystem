import axios from 'axios'

const url = 'https://test.youbaobao.xyz:18081/book/home/v2?openId=1234'
axios.get(url).then(response => {
  console.log(response)
})