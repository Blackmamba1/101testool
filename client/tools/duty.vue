<style rel="stylesheet/stylus" lang="stylus" scoped>
    $active-color = #5eb95e

    .lucky-guy
        color: $active-color
        font-weight: 700

    .weekday, .weekend
        > h3
            margin-top 10px 0
        th, td
            text-align: center
        tr.active
            background: lightness($active-color, 95%)

        td.active
            background: $active-color
            color #fff
            border 0
        td.date
            width 25%
</style>

<template>
    <div>
        <h2 class="content-subhead">
            日期：{{now}} &emsp; <br/>
            今日客服：<span class="lucky-guy">{{todayLuckyGuy}}</span><br/>
            明日客服：<span class="next-lucky-guy">{{tomorrowLuckyGuy}}</span><br/>
        </h2>
        <div class="pure-g">
            <div class="weekday pure-u-1 pure-u-md-1-2">
                <h3>工作日</h3>
                <table class="pure-table pure-table-bordered">
                    <thead>
                    <tr>
                        <th>日期</th>
                        <th>双周</th>
                        <th>单周</th>
                    </tr>
                    </thead>

                    <tbody>
                    <tr v-for="n in 5">
                        <td class="date">{{n+1 | num2Weekday}}</td>
                        <td :class="{'active': todayIsWeekDay && n === todayCoordinate.x && 0 === todayCoordinate.y}">
                            {{dutyOrder[n][0]}}
                        </td>
                        <td :class="{'active': todayIsWeekDay && n === todayCoordinate.x && 1 === todayCoordinate.y}">
                            {{dutyOrder[n][1]}}
                        </td>
                    </tr>
                    </tbody>
                </table>
            </div>

            <div class="weekend pure-u-1 pure-u-md-1-2">
                <h3>双休日</h3>
                <table class="pure-table pure-table-bordered">
                    <thead>
                    <tr>
                        <th>周数</th>
                        <th>周六</th>
                        <th>周日</th>
                    </tr>
                    </thead>

                    <tbody>
                    <tr v-for="n in 5" :class="{'active': currentWeekNumRemainder === (n+1)}">
                        <td class="date">{{n+1 | remainder2WeekNum}}</td>
                        <td :class="{'active': !todayIsWeekDay && n === todayCoordinate.x && 0 === todayCoordinate.y}">
                            {{dutyOrder[n][0]}}
                        </td>
                        <td :class="{'active': !todayIsWeekDay && n === todayCoordinate.x && 1 === todayCoordinate.y}">
                            {{dutyOrder[n][1]}}
                        </td>
                    </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<!--suppress JSUnresolvedFunction -->
<script type="text/babel">
    import moment from 'moment'
    import {isEven} from 'server/utils/lang'
    moment.locale('zh-cn')
    const today = moment()
    /**
     * 是否是工作日
     * @param dayInWeek 一周中的第几天(从 1 开始)
     * @returns {boolean}
     */
    const _isWeekDay = (dayInWeek) => {
        return dayInWeek >= 1 && dayInWeek <= 5
    }

    /**
     * 获得数字除 5 后的余数, 如被整除, 返回 5
     * @param num
     * @returns {number}
     * @private
     */
    const _getRemainderBy5 = (num) => {
        let _remainder = num % 5
        // 如果被 5 整除 , 余数当成 5
        if (_remainder === 0) {
            _remainder = 5
        }
        return _remainder
    }

    export default{
        name: 'duty',
        data() {
            return {
//                dayInWeek: today.isoWeekday(),
                todayIsWeekDay: _isWeekDay(today.isoWeekday()),
                weekInYear: today.isoWeek(),
                now: today.format('YYYY 年 M 月 D 日 dddd'),
                dutyOrder: [
                    [ '许彤', '叶金芳' ],
                    [ '潘永雷', '张经国' ],
                    [ '陈虎', '冯艺 + 孙悦 + ...' ],
                    [ '谭容', '赵欣 + 徐艮权' ],
                    [ '李攀', '李伟' ]
                ],
                todayCoordinate: { x: 0, y: 0 },
                tomorrowCoordinate: { x: 0, y: 0 }
            }
        },
        filters: {
            /**
             * 索引+1 -> 星期几: 1 -> 星期一
             * @param num
             * @returns {*}
             */
            num2Weekday(num) {
                return moment().isoWeekday(num).format('dddd')
            },
            /**
             * 周数
             * @param remainder
             * @returns {number}
             */
            remainder2WeekNum(remainder) {
                return this.weekInYear + (remainder - this.currentWeekNumRemainder)
            }
        },
        computed: {
            currentWeekNumRemainder() {
                return _getRemainderBy5(this.weekInYear)
            },
            todayLuckyGuy() {
                return this.dutyOrder[ this.todayCoordinate.x ][ this.todayCoordinate.y ]
            },
            tomorrowLuckyGuy() {
                return this.dutyOrder[ this.tomorrowCoordinate.x ][ this.tomorrowCoordinate.y ]
            }
        },
        methods: {
            /**
             * 根据日期找到值日人在 dutyOrder 中的坐标
             * @param mDate moment 日期
             * @returns {*}
             */
            getPosByDate(mDate) {
                const coordinate = {}
                const dayInWeek = mDate.isoWeekday()
                const weekInYear = mDate.isoWeek()
                // 工作日
                if (_isWeekDay(dayInWeek)) {
                    // 天转换成索引需减 1
                    coordinate.x = dayInWeek - 1
                    coordinate.y = isEven(weekInYear) ? 0 : 1
                } else {
                    // 周末
                    // 天转换成索引需减 1
                    coordinate.x = _getRemainderBy5(weekInYear) - 1
                    coordinate.y = dayInWeek === 6 ? 0 : 1
                }
                return coordinate
            }
        },
        ready() {
            this.todayCoordinate = this.getPosByDate(today)
            // 注意: 不要使用 today +1 来当成明天用, 这回导致今天被 +1 ...
            this.tomorrowCoordinate = this.getPosByDate(moment().add(1, 'd'))
        }
    }
</script>