<script setup>
import { ref, onMounted } from 'vue';

const winningNumbers = ref({
    first: null,
    nearFirst: [],
    second: [],
    lastTwoDigits: null
});
const isDrawn = ref(false);
const userTicket = ref('');

const checkHistory = ref([]); 

const padNumber = (num, length) => String(num).padStart(length, '0');

const drawPrizes = () => {
    const firstPrizeNum = Math.floor(Math.random() * 1000);
    winningNumbers.value.first = padNumber(firstPrizeNum, 3);

    const nearBefore = (firstPrizeNum === 0) ? 999 : firstPrizeNum - 1;
    const nearAfter = (firstPrizeNum === 999) ? 0 : firstPrizeNum + 1;
    winningNumbers.value.nearFirst = [padNumber(nearBefore, 3), padNumber(nearAfter, 3)];

    const secondPrizes = new Set();
    while (secondPrizes.size < 3) {
        const num = Math.floor(Math.random() * 1000);
        if (num !== firstPrizeNum) {
            secondPrizes.add(padNumber(num, 3));
        }
    }
    winningNumbers.value.second = Array.from(secondPrizes);

    const lastTwoDigitsNum = Math.floor(Math.random() * 100);
    winningNumbers.value.lastTwoDigits = padNumber(lastTwoDigitsNum, 2);
    
    isDrawn.value = true;
    
    checkHistory.value = []; 

    localStorage.setItem('lotteryResults', JSON.stringify(winningNumbers.value));
};

const checkPrize = () => {
    if (!/^\d{3}$/.test(userTicket.value)) {
        alert('กรุณากรอกตัวเลข 3 หลักให้ถูกต้อง');
        return;
    }

    const ticket = userTicket.value;
    const prizesWon = [];

    if (ticket === winningNumbers.value.first) {
        prizesWon.push('ยินดีด้วย! คุณถูกรางวัลที่ 1');
    }
    if (winningNumbers.value.nearFirst.includes(ticket)) {
        prizesWon.push('ยินดีด้วย! คุณถูกรางวัลข้างเคียงรางวัลที่ 1');
    }
    if (winningNumbers.value.second.includes(ticket)) {
        prizesWon.push('ยินดีด้วย! คุณถูกรางวัลที่ 2');
    }
    if (ticket.slice(-2) === winningNumbers.value.lastTwoDigits) {
        prizesWon.push('ยินดีด้วย! คุณถูกรางวัลเลขท้าย 2 ตัว');
    }

    let newResult;
    if (prizesWon.length > 0) {
        newResult = {
            checkedNumber: ticket,
            isWinner: true,
            prizesWon: prizesWon,
            message: ''
        };
    } else {
        newResult = {
            checkedNumber: ticket,
            isWinner: false,
            prizesWon: [],
            message: 'เสียใจด้วย คุณไม่ถูกรางวัล'
        };
    }

    checkHistory.value.unshift(newResult);

    userTicket.value = '';
};
onMounted(() => {
    const savedResults = localStorage.getItem('lotteryResults');
    if (savedResults) {
        winningNumbers.value = JSON.parse(savedResults);
        isDrawn.value = true;
    }
});
</script>

<template>
  <main>
    <div class="container my-5">
      <div class="row justify-content-center">
        <div class="col-lg-10">
          <div class="card prize-card">
            <div class="card-header text-center bg-dark text-white">
              <h2>ระบบสุ่มและตรวจรางวัลล็อตเตอรี่ 3 หลัก</h2>
            </div>
            <div class="card-body">
              <div class="text-center mb-4">
                <button @click="drawPrizes" class="btn btn-danger btn-lg btn-draw">
                  ดำเนินการสุ่มรางวัล
                </button>
              </div>
              <div v-if="isDrawn">
                <h3 class="text-center mb-3">ผลการออกรางวัล</h3>
                <table class="table table-bordered table-striped text-center">
                  <thead class="table-light">
                    <tr>
                      <th>รางวัล</th>
                      <th>เลขที่ออก</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr>
                      <td class="prize-label">รางวัลที่ 1</td>
                      <td class="prize-number">{{ winningNumbers.first }}</td>
                    </tr>
                    <tr>
                      <td class="prize-label">รางวัลข้างเคียงรางวัลที่ 1</td>
                      <td>
                        <span class="fs-4 fw-bold mx-2">{{ winningNumbers.nearFirst[0] }}</span>
                        <span class="fs-4 fw-bold mx-2">{{ winningNumbers.nearFirst[1] }}</span>
                      </td>
                    </tr>
                    <tr>
                      <td class="prize-label">รางวัลที่ 2 (3 รางวัล)</td>
                      <td>
                        <span v-for="num in winningNumbers.second" :key="num" class="fs-4 fw-bold mx-2">{{ num }}</span>
                      </td>
                    </tr>
                    <tr>
                      <td class="prize-label">รางวัลเลขท้าย 2 ตัว</td>
                      <td class="fs-4 fw-bold text-primary">XX{{ winningNumbers.lastTwoDigits }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <div v-else class="text-center text-muted p-4">
                <p>ยังไม่มีการสุ่มรางวัล กรุณากดปุ่ม "ดำเนินการสุ่มรางวัล"</p>
              </div>

              <hr class="my-4">

              <div class="row justify-content-center">
                <div class="col-md-8">
                  <h3 class="text-center mb-3">ตรวจรางวัลของคุณ</h3>
                  <form @submit.prevent="checkPrize">
                    <div class="input-group mb-3">
                      <input 
                          type="text" 
                          v-model="userTicket" 
                          class="form-control form-control-lg" 
                          placeholder="กรอกเลข 3 หลักของคุณ"
                          maxlength="3"
                          pattern="\d{3}"
                          required
                          title="กรุณากรอกตัวเลข 3 หลัก"
                      >
                      <button class="btn btn-success" type="submit" :disabled="!isDrawn">ตรวจรางวัล</button>
                    </div>
                  </form>
                  <div v-if="checkHistory.length > 0" class="mt-3">
                    <h4 class="text-center">ประวัติการตรวจ</h4>
                    <ul class="list-group">
                      <li v-for="(result, index) in checkHistory" :key="index" class="list-group-item">
                        <div :class="['alert', 'mb-0', result.isWinner ? 'alert-success' : 'alert-secondary']">
                          <strong>ผลการตรวจสอบหมายเลข {{ result.checkedNumber }}:</strong>
                          
                          <ul v-if="result.isWinner" class="mb-0">
                            <li v-for="prize in result.prizesWon" :key="prize">{{ prize }}</li>
                          </ul>

                          <p v-if="!result.isWinner" class="mb-0 fw-bold">{{ result.message }}</p>
                        </div>
                      </li>
                    </ul>
                  </div>

                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
.prize-card {
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    transition: transform 0.2s;
}
.prize-card:hover {
    transform: translateY(-5px);
}
.prize-number {
    font-size: 2.5rem;
    font-weight: bold;
    color: #dc3545;
}
.prize-label {
    font-weight: 500;
    color: #6c757d;
}
.btn-draw {
    padding: 10px 20px;
    font-size: 1.2rem;
}
</style>