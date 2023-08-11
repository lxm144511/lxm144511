- 👋 Hi, I’m @lxm144511
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

package main

import (
	"encoding/json"
	"fmt"
	"os"
	"regexp"
	"strconv"
	"strings"
	"time"
)

// cpu核数
func CpuCore() interface{} {
	// 定义cpu 核数初始值
	cpucores := 0
	// 读取/proc/cpuinfo 文件内容
	file, err := os.ReadFile("/proc/cpuinfo")
	if err != nil {
		fmt.Println("open  file  failed ", err)
	}
	// 使用正则表达式切割多个空格
	reg := regexp.MustCompile(`\s+`)
	newslice := strings.Split(string(file), "\n")
	//fmt.Println(len(newslice))
	for _, values := range newslice {
		// 匹配多个进行切割，生成切片
		results := reg.Split(values, -1)
		if results[0] == "processor" && results != nil {
			cpucores += 1

		}

	}
	return cpucores
}

// cpu使用率
func CpuUsage() (totalCpuTime, idle uint64) {
	//读取 proc/state 文件信息
	fire, err := os.ReadFile("/proc/stat")
	if err != nil {
		fmt.Println("open /proc/stat file failed ", err)
	}
	// 获取文件第一行信息
	//1. 读取每一行数据安装换行符进行分割，生成切片
	for _, values := range strings.Split(string(fire), "\n") {
		//2. 对每一行数据多余的空行进行分割
		reg := regexp.MustCompile(`\s+`)
		results := reg.Split(string(values), -1)
		if results[0] == "cpu" {
			//totalCpuTime  =  user  +  nice + system + idle + iowait + irq + softirq + stealstolen + guest + guest_nice
			user, _ := strconv.ParseUint(results[1], 10, 64)
			nice, _ := strconv.ParseUint(results[2], 10, 64)
			system, _ := strconv.ParseUint(results[3], 10, 64)
			idle, _ := strconv.ParseUint(results[4], 10, 64)
			iowait, _ := strconv.ParseUint(results[5], 10, 64)
			irq, _ := strconv.ParseUint(results[6], 10, 64)
			softirq, _ := strconv.ParseUint(results[7], 10, 64)
			stealstolen, _ := strconv.ParseUint(results[8], 10, 64)
			guest, _ := strconv.ParseUint(results[9], 10, 64)
			guest_nice, _ := strconv.ParseUint(results[10], 10, 64)

			//2.cpu使用率计算：
			//请在一段时间内（推荐：必须大于0s，小于等于1s），获取两次cpu时间分配信息。
			//计算两次的cpu总时间：total_2 - total_1
			//计算两次的cpu剩余时间：idle_2 - idle_1
			//计算两次的cpu使用时间：used = (total_2 - total_1) - (idle_2 - idle_1)
			//cpu使用率 = 使用时间 / 总时间  100% = used / total  100%
			//Average idle time (%) = (idle * 100) / (user + nice + system + idle + iowait + irq + softirq + steal + guest + guest_nice)
			//totalCpuTime  =  user  +  nice + system + idle + iowait + irq + softirq + stealstolen + guest + guest_nice
			totalCpuTime := user + nice + system + idle + iowait + irq + softirq + stealstolen + guest + guest_nice
			//Average_idle_time := idle * 100 / totalCpuTime
			//fmt.Println(Average_idle_time)
			return totalCpuTime, idle
		}

	}
	return

}

func main() {
	// 创建map ,将cpu信息存入到map中去
	cpuinfo := make(map[string]interface{})

	//fmt.Println(cpucores)
	totalCpuTime01, idle01 := CpuUsage()
	//fmt.Println(totalCpuTime01, idle01)
	time.Sleep(time.Second * 1)
	totalCpuTime02, idle02 := CpuUsage()
	cpucores := CpuCore()
	// 计算cpu使用率
	idleTicks := float64(idle02 - idle01)
	totalCpuTimeTicks := float64(totalCpuTime02 - totalCpuTime01)
	cpuUsage := 100 * (totalCpuTimeTicks - idleTicks) / totalCpuTimeTicks
	cpuinfo["cpuCores"] = cpucores
	// strconv.FormatFloat() 方法将float类型转换为string 类型 , prec : 表示为保留小数 bitsize : 表示float64或者float64
	cpuinfo["cpuUsage"] = strconv.FormatFloat(cpuUsage, 'f', 2, 64) + "%"
	cpuinfostr, _ := json.Marshal(cpuinfo)
	fmt.Println(string(cpuinfostr))

}
