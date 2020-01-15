<template>
    <div class="home">
        <p>Your best score : {{ highScore }}</p>
        <p>Your score: {{ score }}</p>
        <div class="board">
            <div v-for="(i, row) in area" :key="`row-${row}`" class="g-row">
                <div
                        :id="col + ' ' + row"
                        v-for="(j, col) in area[row]"
                        :key="`col-${col}`"
                        style="display: inline-block;border:black solid 1px;width: 10%;height: 50px"
                >
                    <div
                            :class="['g-col', 'g-ball-' + area[row][col]]"
                            @click="handleBallClick(row, col)"
                    ></div>
                </div>
            </div>
        </div>
        <button @click="setRandomBalls">Set balls</button>
        <button @click="restart">Restart</button>
    </div>
</template>

<script>

    export default {
        name: 'home',
        components: {},
        data() {
            return {
                area: [],
                selected_ball: null,
                highScore: 0,
                score: 0,
                status: 'ALIVE'
            }
        },
        created() {
            this.initArea();
            this.setRandomBalls();
        },
        methods: {

            restart() {
                this.status = 'ALIVE';
                this.initArea();
                this.setRandomBalls();
                if (this.score > this.highScore)
                    this.highScore = this.score;
                this.score = 0;
            },

            getRandom(min, max) {
                min = Math.ceil(min);
                max = Math.floor(max);
                return Math.floor(Math.random() * (max - min + 1)) + min;
            },

            checkWall(x1, y1, map) {
                map[x1][y1] = 1;
                if (x1 + 1 < 9 && map[x1 + 1][y1] != 'w' && map[x1 + 1][y1] != 1) {
                    map = this.checkWall(x1 + 1, y1, map);
                }
                if (y1 + 1 < 9 && map[x1][y1 + 1] != 'w' && map[x1][y1 + 1] != 1) {
                    map = this.checkWall(x1, y1 + 1, map);
                }
                if (y1 - 1 >= 0 && map[x1][y1 - 1] != 'w' && map[x1][y1 - 1] != 1) {
                    map = this.checkWall(x1, y1 - 1, map);
                }
                if (x1 - 1 >= 0 && map[x1 - 1][y1] != 'w' && map[x1 - 1][y1] != 1) {
                    map = this.checkWall(x1 - 1, y1, map);
                }
                return map;
            },

            moveBalls(x1, y1, x2, y2) {
                let arr = [];
                for (let i = 0; i < 9; i++) {
                    for (let j = 0; j < 9; j++) {
                        if (!arr[i])
                            arr[i] = [];
                        if (this.area[i][j] != 0) {
                            arr[i][j] = 'w';
                        } else {
                            arr[i][j] = 0
                        }

                    }
                }
                arr[x1][y1] = 1;

                if (this.area[x1][y1] == 0) return false;
                if (this.area[x2][y2] != 0) return false;

                let map = this.checkWall(x1, y1, arr);
                if (map[x2][y2] == 1) {
                    this.area[x2][y2] = this.area[x1][y1];
                    this.area[x1][y1] = 0;
                    return true;
                }
            },

            checkLine() {
                let count = 0;
                for (let i = 0; i < 9; i++) {
                    let value = 0;
                    for (let j = 0; j < 9; j++) {
                        if (this.area[i][j] == value) {
                            count++;
                        } else {
                            if (count > 4 && value != 0) {
                                for (let k = j - count; k < j; k++) {
                                    this.area[i][k] = 0;
                                    this.$forceUpdate();
                                }
                                this.score += count;
                            }
                            value = this.area[i][j];
                            count = 1;
                        }
                    }
                    if (count > 4 && value != 0) {
                        for (let k = 9 - count; k <= 8; k++) {
                            this.area[i][k] = 0;
                            this.$forceUpdate();
                        }
                        this.score += count;
                    }
                }
                count = 0;
                for (let i = 0; i < 9; i++) {
                    let value = 0;
                    for (let j = 0; j < 9; j++) {
                        if (this.area[j][i] == value) {
                            count++;
                        } else {
                            if (count > 4 && value != 0) {
                                for (let k = j - count; k < j; k++) {
                                    this.area[k][i] = 0;
                                    this.$forceUpdate();
                                }
                                this.score += count;
                            }
                            value = this.area[j][i];
                            count = 1;
                        }
                    }
                    if (count > 4 && value != 0) {
                        for (let k = 9 - count; k < 9; k++) {
                            this.area[k][i] = 0;
                            this.$forceUpdate();
                        }
                        this.score += count;
                    }
                }
            },

            example() {
                for (let k = 8; k >= 0; k--) {
                    for (let i = 0; i <= k; i++) {
                        setTimeout(() => {
                            this.area[i][k - i] = k + 1;
                            this.$forceUpdate()
                        }, k * 500)
                    }
                }
            },

            searchForAscendingLines() {
                let countD = 1;
                for (let i = 0; i < 9; i++) {
                    let value = 0;
                    for (let j = 0; j + i < 9; j++) {
                        if (value == this.area[j + i][j]) {
                            countD++;
                        } else {
                            if (countD > 4 && value != 0) {
                                for (let k = i + j - countD, h = j - countD; k < i + j; h++, k++) {
                                    this.area[k][h] = 0;
                                    this.$forceUpdate();
                                }
                                this.score += countD;
                            }
                            value = this.area[j + i][j];
                            countD = 1;
                        }
                    }
                    if (countD > 4 && value != 0) {
                        for (let k = i, h = 0; k < 9; h++, k++) {
                            this.area[k][h] = 0;
                            this.$forceUpdate();
                        }
                        this.score += countD;
                    }
                    countD = 1;
                }

                countD = 1;
                let array = [1,2,3];
                array.forEach()
                for (let i = 0; i < 9; i++) {
                    let value = 0;
                    for (let j = 0; j + i < 9; j++) {
                        if (value == this.area[j][j + i]) {
                            countD++;
                        } else {
                            if (countD > 4 && value != 0) {
                                for (let k = j - countD, h = i + j - countD; k < 8 - i; h++, k++) {
                                    this.area[k][h] = 0;
                                    this.$forceUpdate();
                                }
                                this.score += countD;
                            }
                            value = this.area[j][j + i];
                            countD = 1;
                        }
                    }
                    if (countD > 4 && value != 0) {
                        for (let k = i, h = 0; k < 9; h++, k++) {
                            this.area[h][k] = 0;
                            this.$forceUpdate();
                        }
                        this.score += countD;
                    }
                    countD = 1;
                }
            },

            searchDescLines() {
                let countD = 1;
                for (let i = 8; i >= 0; i--) {
                    let value = 0;
                    for (let j = 0; j <= i; j++) {
                        if (value == this.area[j][i - j]) {
                            countD++;
                        } else {
                            if (countD > 4 && value != 0) {
                                for (let k = j - 1, h = i - k; k > j - countD - 1; k--, h++) {
                                    this.area[k][h] = 0;
                                    this.$forceUpdate();
                                }
                                this.score += countD;
                            }
                            value = this.area[j][i - j];
                            countD = 1;
                        }
                    }
                    if (countD > 4 && value != 0) {
                        for (let k = 0, h = i; k < countD; h--, k++) {
                            this.area[h][k] = 0;
                            this.$forceUpdate()
                        }
                        this.score += countD;
                    }
                    countD = 1;
                }
                countD = 1;
                for (let k = 0; k < 9; k++) {
                    let value = 0;
                    for (let i = 8, j = k; i >= k; j++, i--) {
                        if (value == this.area[i][j]) {
                            countD++;
                        } else {
                            if (countD > 4 && value != 0) {
                                for (let x = i + countD, y = j - countD; y < j; x--, y++) {
                                    this.area[x][y] = 0;
                                    this.$forceUpdate();
                                }
                                this.score += countD;
                            }
                            value = this.area[i][j];
                            countD = 1;
                        }
                    }

                    if (countD > 4 && value != 0) {
                        for (let x = k + countD - 1, y = 8 - countD + 1; y <= 8; x--, y++) {
                            this.area[x][y] = 0;
                            this.$forceUpdate();
                        }
                        this.score += countD;
                    }
                    countD = 1;
                }
            },

            removeLinesX(row, beg, end) {
                for (let i = beg; i < end; i++) {
                    this.area[i][row] = 0;
                    this.$forceUpdate();
                }
            },

            addTestBalls() {
                this.area[3][2] = 1;
                this.area[4][3] = 1;
                this.area[5][4] = 1;
                this.area[6][5] = 1;
                this.area[7][6] = 1;

                // this.example()
                // this.searchDescLines()
            },

            setRandomBall() {
                var setted = false

                if (this.getFreeCelsCount() < 3) {
                    this.status = 'LOSE'
                    // eslint-disable-next-line no-console
                    console.log('GAME OVER!')
                    // eslint-disable-next-line no-console
                    console.log('Free: ' + this.getFreeCelsCount())
                    return
                }

                while (!setted) {
                    let i = this.getRandom(0, 8);
                    let j = this.getRandom(0, 8);
                    if (this.area[i][j] == 0) {
                        this.area[i][j] = this.getRandom(1, 6);
                        setted = true;
                    }
                }

            },

            getFreeCelsCount() {
                var count = 0;
                for (var i = 0; i < 9; i++) {
                    for (var j = 0; j < 9; j++) {
                        if (this.area[i][j] == 0)
                            count++;
                    }
                }

                return count
            },

            setRandomBalls() {
                if (this.status == 'LOSE') {
                    alert('Game over');
                    return
                }
                this.setRandomBall();
                this.setRandomBall();
                this.setRandomBall();
                this.searchForAscendingLines();
                this.searchDescLines();
                this.checkLine();
                this.$forceUpdate();
            },

            initArea() {
                this.area = [];
                for (var i = 0; i < 9; i++) {
                    for (var j = 0; j < 9; j++) {
                        if (!this.area[i])
                            this.area[i] = [];
                        this.area[i][j] = 0
                    }
                }
                this.$forceUpdate();
            },

            handleBallClick(x, y) {
                if (this.status == 'LOSE') {
                    alert('Game over');
                    return
                }
                if (!this.selected_ball && this.area[x][y] != 0) {
                    this.selected_ball = [x, y];
                    document.getElementById(y + ' ' + x).style.background = 'cadetblue';
                } else {
                    if (this.area[x][y] == 0) {
                        if (this.moveBalls(
                            this.selected_ball[0],
                            this.selected_ball[1],
                            x, y
                        )) {
                            document.getElementById(this.selected_ball[1] + ' ' + this.selected_ball[0]).style.background = 'white';
                            this.selected_ball = null;
                            this.setRandomBalls();
                            this.$nextTick(() => {
                                this.$forceUpdate()
                            })
                        }
                    } else {
                        document.getElementById(this.selected_ball[1] + ' ' + this.selected_ball[0]).style.background = 'white';
                        this.selected_ball = null;
                        this.selected_ball = [x, y];
                        document.getElementById(y + ' ' + x).style.background = 'cadetblue';
                    }
                }
            }
        }
    }
</script>

<style>
    .board {
        margin: auto;
        width: 30%;
        border-style: solid;
        border-color: black;
    }

    .g-row {

    }

    .g-col {
        display: inline-block;
        text-align: center;
        width: 100%;
        height: 100%;
        border-radius: 50%;
    }

    .g-ball-1 {
        background: red;
    }

    .g-ball-2 {
        background: green;
    }

    .g-ball-3 {
        background: blue;
    }

    .g-ball-4 {
        background: yellow;
    }

    .g-ball-5 {
        background: orange;
    }

    .g-ball-6 {
        background: deepskyblue;
    }
</style>
