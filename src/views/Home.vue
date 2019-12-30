<template>
    <div class="home">
        <div v-for="(i, row) in area" :key="`row-${row}`" class="g-row">
            <div
                    v-for="(j, col) in area[row]"
                    :key="`col-${col}`"
                    :class="['g-col', 'g-ball-' + area[row][col]]"
                    @click="handleBallClick(row, col)"
            >
                {{area[row][col]}}
            </div>
        </div>

        <button @click="setRandomBalls">Set balls</button>
    </div>
</template>

<script>

    export default {
        name: 'home',
        components: {},
        data() {
            return {
                area: [],
                selected_ball: null
            }
        },
        created() {
            this.initArea()
            this.setRandomBalls()
        },
        methods: {
            getRandom(min, max) {
                min = Math.ceil(min)
                max = Math.floor(max)
                return Math.floor(Math.random() * (max - min + 1)) + min
            },

            moveBalls(x1, y1, x2, y2) {

                if (this.area[x1][y1] == 0) return false
                if (this.area[x2][y2] != 0) return false

                this.area[x2][y2] = this.area[x1][y1]
                this.area[x1][y1] = 0

                this.checkLine(x2, y2);
                return true
            },

            checkLine(x, y) {
                let value = this.area[x][y]
                let countX = 0;
                let countY = 0;
                for (let i = 0; i < 9; i++) {
                    if (this.area[i][y] == value) {
                        countY++;
                    } else {
                        if (countY >= 5) {
                            this.removeLinesX(y, i - countY, i);
                        }
                        countY = 0;
                    }
                }
                if (countY >= 5) {
                    this.removeLinesX(y, 9 - countY, 9);
                }
                for (let i = 0; i < 9; i++) {
                    if (this.area[x][i] == value) {
                        countX++;
                    } else {
                        if (countX >= 5) {
                            for (let j = i - countX; j < i; j++) {
                                this.area[x][j] = 0;
                                this.$forceUpdate()
                            }
                        }
                        countX = 0;
                    }
                }
                if (countX >= 5) {
                    for (let j = 8 - countX; j < 9; j++) {
                        this.area[x][j] = 0;
                        this.$forceUpdate()
                    }
                }

            },

            example() {
                for (let i = 0; i < 9; i++) {
                    // let value = 0;
                    for (let j = 0; j + i < 9; j++) {
                        setTimeout(() => {
                            this.area[j + i][j] = i + 1
                            this.$forceUpdate()
                        }, i * 500)
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
                                    this.$forceUpdate()
                                }
                            }
                            value = this.area[j + i][j];
                            countD = 1;
                        }
                    }
                    if (countD > 4 && value != 0) {
                        for (let k = i, h = 0; k < 9; h++, k++) {
                            this.area[k][h] = 0;
                            this.$forceUpdate()
                        }
                    }
                    countD = 1;
                }

                countD = 1;
                for (let i = 0; i < 9; i++) {
                    let value = 0;
                    for (let j = 0; j + i < 9; j++) {
                        if (value == this.area[j][j + i]) {
                            countD++;
                        } else {
                            if (countD > 4 && value != 0) {
                                for (let k = j - countD, h = i + j - countD; k < 8 - i; h++, k++) {
                                    this.area[k][h] = 0;
                                    this.$forceUpdate()
                                }
                            }
                            value = this.area[j][j + i];
                            countD = 1;
                        }
                    }
                    if (countD > 4 && value != 0) {
                        for (let k = i, h = 0; k < 9; h++, k++) {
                            this.area[h][k] = 0;
                            this.$forceUpdate()
                        }
                    }
                    countD = 1;
                }
            },

            searchDescLines() {
                // for (let i = 0; i < 9; i++) {
                // let value = 0;
                // for (let j = 0; j + i < 9; j++) {
                //     setTimeout(() => {
                //         this.area[j + i][j] = i + 1
                //         this.$forceUpdate()
                //     }, i * 500)
                // }
                // }

                let countD = 1;
                for (let i = 8; i >= 0; i--) {
                    let value = 0;
                    for (let j = 0; j <= i; j++) {
                        if (value == this.area[j][i - j]) {
                            countD++;
                        } else {
                            if (countD > 4 && value != 0) {
                                for (let k = j - 1, h = i - j + 1; k > j - countD - 1; k--, h++) {
                                    this.area[k][h] = 0;
                                    this.$forceUpdate()
                                }
                            }
                            value = this.area[j][i - j];
                            countD = 1;
                        }
                    }
                    if (countD > 4 && value != 0) {
                        // eslint-disable-next-line no-console
                        console.log('DELETE'); // TODO delete me
                        for (let k = i, h = 0; k > i - countD; h++, k--) {
                            // eslint-disable-next-line no-console
                            console.log('k ', k); // TODO delete me
                            // eslint-disable-next-line no-console
                            console.log('h ', h); // TODO delete me
                            this.area[h][k] = 0;
                            this.$forceUpdate()
                        }
                    }
                }

            },

            removeLinesX(row, beg, end) {
                for (let i = beg; i < end; i++) {
                    this.area[i][row] = 0;
                    this.$forceUpdate()
                }
            },

            addTestBalls() {
                this.area[3][2] = 1
                this.area[4][3] = 1
                this.area[5][4] = 1
                this.area[6][5] = 1
                this.area[7][6] = 1

                // this.example()
                this.searchDescLines()
            },

            setRandomBall() {
                // eslint-disable-next-line no-console
                console.log('');
                var setted = false

                if (this.getFreeCelsCount() < 3) {
                    // eslint-disable-next-line no-console
                    console.log('GAME OVER!')
                    // eslint-disable-next-line no-console
                    console.log('Free: ' + this.getFreeCelsCount())
                    return
                }

                while (!setted) {
                    let i = this.getRandom(0, 8)
                    let j = this.getRandom(0, 8)
                    if (this.area[i][j] == 0) {
                        this.area[i][j] = this.getRandom(1, 4)
                        setted = true
                    }
                }

            },

            getFreeCelsCount() {
                var count = 0;
                for (var i = 0; i < 9; i++) {
                    for (var j = 0; j < 9; j++) {
                        if (this.area[i][j] == 0)
                            count++
                    }
                }

                return count
            },

            setRandomBalls() {
                this.setRandomBall()
                this.setRandomBall()
                this.setRandomBall()
                this.searchForAscendingLines();
                this.searchDescLines();
                this.$forceUpdate()
            },

            initArea() {
                this.area = []
                for (var i = 0; i < 9; i++) {
                    for (var j = 0; j < 9; j++) {
                        if (!this.area[i])
                            this.area[i] = []
                        this.area[i][j] = 0
                    }
                }

                this.addTestBalls()
                this.$forceUpdate()
            },

            handleBallClick(x, y) {
                // eslint-disable-next-line no-console
                console.log(`${x} / ${y}`)

                if (!this.selected_ball) {
                    this.selected_ball = [x, y]
                } else {
                    if (this.moveBalls(
                        this.selected_ball[0],
                        this.selected_ball[1],
                        x, y
                    )) {
                        this.selected_ball = null
                        this.setRandomBalls()
                        this.$nextTick(() => {
                            this.$forceUpdate()
                        })
                    } else {
                        // eslint-disable-next-line no-console
                        console.log('Can\'t move!')
                    }
                }
            }
        }
    }
</script>

<style>
    .g-row {

    }

    .g-col {
        display: inline-block;
        text-align: center;
        width: 20px;
        height: 20px;
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
</style>
