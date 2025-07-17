<template>
  <section class="han_analytics">
    <header>
      <div class="main">
        <div class="logo">
          <img src="./assets/favicon.ico">
          <span>undefined's Blog User Analytics</span>
        </div>
        <h2>一个托管在 Cloudflare Pages 上的免费开源网站分析仪表板</h2>
      </div>
    </header>
    <main>
      <header>
        <Alert>
          <AlertDescription>
            <p>· 这是一个简单的网络分析跟踪器和仪表板，托管在 Cloudflare 上,无成本稳定运行,每天可达10万次免费统计。</p>
            <p style="font-weight: bold;">· 项目开源地址: <a class="git-link" href="https://github.com/uxiaohan/HanAnalytics"
                target="_blank">Han-Analytics</a>
            </p>
            <p style="font-weight: bold;">· 博客主站点: <a class="git-link" href="https://undefined.today"
                target="_blank">undefined's Blog</a>
            </p>
          </AlertDescription>
        </Alert>
      </header>

      <section class="main">
        <div class="pb-5 grid md:grid-cols-2 sm:grid-cols-1 items-start">
          <div class="flex gap-[16px] pb-6">
            <div class="w-3/6">
              <Select :disabled="siteList.length < 1 || getDatasStatus" v-model="siteValue"
                @update:model-value="getDatas">
                <SelectTrigger class="w-[218px]">
                  <SelectValue placeholder="选择站点" />
                </SelectTrigger>
                <SelectContent>
                  <SelectGroup>
                    <SelectLabel>网页列表</SelectLabel>
                    <SelectItem :value="i" v-for="i in siteList" :key="i">{{ i }}</SelectItem>
                  </SelectGroup>
                </SelectContent>
              </Select>
            </div>
            <div class="w-3/6">
              <Select :disabled="siteList.length < 1 || getDatasStatus" v-model="timeValue"
                @update:model-value="getDatas">
                <SelectTrigger class="w-[218px]">
                  <SelectValue placeholder="选择周期" />
                </SelectTrigger>
                <SelectContent>
                  <SelectGroup>
                    <SelectLabel>时间周期</SelectLabel>
                    <SelectItem :value="i.value" v-for="i in timeList" :key="i.name">{{ i.name }}</SelectItem>
                  </SelectGroup>
                </SelectContent>
              </Select>
            </div>
          </div>
          <div
            class="flex justify-end text-center md:text-right line-clamp-1 [&>.views-item]:flex [&>.views-item]:flex-col [&>.views-item]:items-center md:[&>.views-item]:items-end [&>.views-item]:gap-4 [&>.views-item>span]:text-sm [&>.views-item>p]:text-3xl [&>.views-item>p]:line-clamp-1 [&>.views-item>p]:w-full">
            <div class="views-item w-full overflow-hidden">
              <span>浏览量</span>
              <div class="space-y-2 w-[50%]" v-if="resData.visit.views === undefined">
                <Skeleton class="h-4  w-[50%] ml-auto" />
                <Skeleton class="h-4" />
              </div>
              <p v-else>{{ resData.visit.views }}</p>
            </div>
            <div class="views-item w-full overflow-hidden">
              <span>访客</span>
              <div class="space-y-2 w-[50%]" v-if="resData.visit.visitor === undefined">
                <Skeleton class="h-4  w-[50%] ml-auto" />
                <Skeleton class="h-4" />
              </div>
              <p v-else>{{ resData.visit.visitor }}</p>
            </div>
            <div class="views-item w-full overflow-hidden">
              <span>访问量</span>
              <div class="space-y-2 w-[50%]" v-if="resData.visit.visit === undefined">
                <Skeleton class="h-4  w-[50%] ml-auto" />
                <Skeleton class="h-4" />
              </div>
              <p v-else>{{ resData.visit.visit }}</p>
            </div>
          </div>
        </div>

        <div ref="echartsDOM" class="data-view"></div>

        <div class="pt-20 grid md:grid-cols-2 sm:grid-cols-1 gap-[16px]">
          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>网页</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.path != undefined">
                <p class="page-item" v-for="(i, idx) in resData.path" :key="idx">
                  <span class="line-clamp-1">{{ i.name }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>

          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>来源</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.referrer != undefined">
                <p class="page-item" v-for="(i, idx) in resData.referrer" :key="idx">
                  <img v-if="i.name" :src="getIconUrl(i.name)">
                  <a :href="i.name" target="_blank" rel="noopener noreferrer" class="line-clamp-1 cursor-pointer">
                    {{ i.name || '(None)' }}
                  </a>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>
        </div>

        <div class="pt-6 grid xl:grid-cols-3 gap-[16px] md:grid-cols-2 sm:grid-cols-1">
          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>浏览器</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.soft != undefined">
                <p class="page-item" v-for="(i, idx) in resData.soft" :key="idx">
                  <span class="line-clamp-1">{{ i.name }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>

          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>操作系统</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.os != undefined">
                <p class="page-item" v-for="(i, idx) in resData.os" :key="idx">
                  <img class="os" :src="getIcon(i.name)">
                  <span class="line-clamp-1">{{ i.name }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>


          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>地区</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.area != undefined">
                <p class="page-item" v-for="(i, idx) in resData.area" :key="idx">
                  <img :src="getIcon(i.name)">
                  <span class="line-clamp-1">{{ i.code }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>
        </div>
      </section>
    </main>
    <footer>
<!--      cf统计-->

      <script type="text/javascript"
              src="https://cdn.staticfile.net/jquery/3.4.1/jquery.min.js"></script>
      <script>
        getCDNinfo = function () {
          $.ajax({
            url: "/cdn-cgi/trace",
            success: function (data, status) {
              let areas = "Antananarivo, Madagascar - (TNR);Cape Town, South Africa - (CPT);Casablanca, Morocco - (CMN);Dar Es Salaam, Tanzania - (DAR);Djibouti City, Djibouti - (JIB);Durban, South Africa - (DUR);Johannesburg, South Africa - (JNB);Kigali, Rwanda - (KGL);Lagos, Nigeria - (LOS);Luanda, Angola - (LAD);Maputo, MZ - (MPM);Mombasa, Kenya - (MBA);Port Louis, Mauritius - (MRU);Réunion, France - (RUN);Bangalore, India - (BLR);Bangkok, Thailand - (BKK);Bandar Seri Begawan, Brunei - (BWN);Cebu, Philippines - (CEB);Chengdu, China - (CTU);Chennai, India - (MAA);Chittagong, Bangladesh - (CGP);Chongqing, China - (CKG);Colombo, Sri Lanka - (CMB);Dhaka, Bangladesh - (DAC);Dongguan, China - (SZX);Foshan, China - (FUO);Fuzhou, China - (FOC);Guangzhou, China - (CAN);Hangzhou, China - (HGH);Hanoi, Vietnam - (HAN);Hengyang, China - (HNY);Ho Chi Minh City, Vietnam - (SGN);Hong Kong - (HKG);Hyderabad, India - (HYD);Islamabad, Pakistan - (ISB);Jakarta, Indonesia - (CGK);Jinan, China - (TNA);Karachi, Pakistan - (KHI);Kathmandu, Nepal - (KTM);Kolkata, India - (CCU);Kuala Lumpur, Malaysia - (KUL);Lahore, Pakistan - (LHE);Langfang, China - (NAY);Luoyang, China - (LYA);Macau - (MFM);Malé, Maldives - (MLE);Manila, Philippines - (MNL);Mumbai, India - (BOM);Nagpur, India - (NAG);Nanning, China - (NNG);New Delhi, India - (DEL);Osaka, Japan - (KIX);Phnom Penh, Cambodia - (PNH);Qingdao, China - (TAO);Seoul, South Korea - (ICN);Shanghai, China - (SHA);Shenyang, China - (SHE);Shijiazhuang, China - (SJW);Singapore, Singapore - (SIN);Suzhou, China - (SZV);Taipei - (TPE);Thimphu, Bhutan - (PBH);Tianjin, China - (TSN);Tokyo, Japan - (NRT);Ulaanbaatar, Mongolia - (ULN);Vientiane, Laos - (VTE);Wuhan, China - (WUH);Wuxi, China - (WUX);Xi'an, China - (XIY);Yerevan, Armenia - (EVN);Zhengzhou, China - (CGO);Zuzhou, China - (CSX);Amsterdam, Netherlands - (AMS);Athens, Greece - (ATH);Barcelona, Spain - (BCN);Belgrade, Serbia - (BEG);Berlin, Germany - (TXL);Brussels, Belgium - (BRU);Bucharest, Romania - (OTP);Budapest, Hungary - (BUD);Chișinău, Moldova - (KIV);Copenhagen, Denmark - (CPH);Cork, Ireland -  (ORK);Dublin, Ireland - (DUB);Düsseldorf, Germany - (DUS);Edinburgh, United Kingdom - (EDI);Frankfurt, Germany - (FRA);Geneva, Switzerland - (GVA);Gothenburg, Sweden - (GOT);Hamburg, Germany - (HAM);Helsinki, Finland - (HEL);Istanbul, Turkey - (IST);Kyiv, Ukraine - (KBP);Lisbon, Portugal - (LIS);London, United Kingdom - (LHR);Luxembourg City, Luxembourg - (LUX);Madrid, Spain - (MAD);Manchester, United Kingdom - (MAN);Marseille, France - (MRS);Milan, Italy - (MXP);Moscow, Russia - (DME);Munich, Germany - (MUC);Nicosia, Cyprus - (LCA);Oslo, Norway - (OSL);Paris, France - (CDG);Prague, Czech Republic - (PRG);Reykjavík, Iceland - (KEF);Riga, Latvia - (RIX);Rome, Italy - (FCO);Saint Petersburg, Russia - (LED);Sofia, Bulgaria - (SOF);Stockholm, Sweden - (ARN);Tallinn, Estonia - (TLL);Thessaloniki, Greece - (SKG);Vienna, Austria - (VIE);Vilnius, Lithuania - (VNO);Warsaw, Poland - (WAW);Zagreb, Croatia - (ZAG);Zürich, Switzerland - (ZRH);Arica, Chile - (ARI);Asunción, Paraguay - (ASU);Bogotá, Colombia - (BOG);Buenos Aires, Argentina - (EZE);Curitiba, Brazil - (CWB);Fortaleza, Brazil - (FOR);Guatemala City, Guatemala - (GUA);Lima, Peru - (LIM);Medellín, Colombia - (MDE);Panama City, Panama - (PTY);Porto Alegre, Brazil - (POA);Quito, Ecuador - (UIO);Rio de Janeiro, Brazil - (GIG);São Paulo, Brazil - (GRU);Santiago, Chile - (SCL);Willemstad, Curaçao - (CUR);St. George's, Grenada - (GND);Amman, Jordan - (AMM);Baghdad, Iraq - (BGW);Baku, Azerbaijan - (GYD);Beirut, Lebanon - (BEY);Doha, Qatar - (DOH);Dubai, United Arab Emirates - (DXB);Kuwait City, Kuwait - (KWI);Manama, Bahrain - (BAH);Muscat, Oman - (MCT);Ramallah - (ZDM);Riyadh, Saudi Arabia - (RUH);Tel Aviv, Israel - (TLV);Ashburn, VA, United States - (IAD);Atlanta, GA, United States - (ATL);Boston, MA, United States - (BOS);Buffalo, NY, United States - (BUF);Calgary, AB, Canada - (YYC);Charlotte, NC, United States - (CLT);Chicago, IL, United States - (ORD);Columbus, OH, United States - (CMH);Dallas, TX, United States - (DFW);Denver, CO, United States - (DEN);Detroit, MI, United States - (DTW);Honolulu, HI, United States - (HNL);Houston, TX, United States - (IAH);Indianapolis, IN, United States - (IND);Jacksonville, FL, United States - (JAX);Kansas City, MO, United States - (MCI);Las Vegas, NV, United States - (LAS);Los Angeles, CA, United States - (LAX);McAllen, TX, United States - (MFE);Memphis, TN, United States - (MEM);Mexico City, Mexico - (MEX);Miami, FL, United States - (MIA);Minneapolis, MN, United States - (MSP);Montgomery, AL, United States - (MGM);Montréal, QC, Canada - (YUL);Nashville, TN, United States - (BNA);Newark, NJ, United States - (EWR);Norfolk, VA, United States - (ORF);Omaha, NE, United States - (OMA);Philadelphia, United States - (PHL);Phoenix, AZ, United States - (PHX);Pittsburgh, PA, United States - (PIT);Port-Au-Prince, Haiti - (PAP);Portland, OR, United States - (PDX);Queretaro, MX, Mexico - (QRO);Richmond, Virginia - (RIC);Sacramento, CA, United States - (SMF);Salt Lake City, UT, United States - (SLC);San Diego, CA, United States - (SAN);San Jose, CA, United States - (SJC);Saskatoon, SK, Canada - (YXE);Seattle, WA, United States - (SEA);St. Louis, MO, United States - (STL);Tampa, FL, United States - (TPA);Toronto, ON, Canada - (YYZ);Vancouver, BC, Canada - (YVR);Tallahassee, FL, United States - (TLH);Winnipeg, MB, Canada - (YWG);Adelaide, SA, Australia - (ADL);Auckland, New Zealand - (AKL);Brisbane, QLD, Australia - (BNE);Melbourne, VIC, Australia - (MEL);Noumea, New caledonia - (NOU);Perth, WA, Australia - (PER);Sydney, NSW, Australia - (SYD)".split(";");
              let area = data.split("colo=")[1].split("\n")[0];
              let areaCode = data.split("colo=")[1].split("\n")[0];
              let tlsVersion = data.split("tls=")[1].split("\n")[0];
              let httpVersion = data.split("http=")[1].split("\n")[0];
              let sniValue = data.split("sni=")[1].split("\n")[0];
              let ipValue = data.split("ip=")[1].split("\n")[0];
              let userAgent = data.split("uag=")[1].split("\n")[0];

              for (var i = 0; i < areas.length; i++) {
                if (areas[i].indexOf(area) != -1) {
                  document.getElementById("cdn").innerHTML = areas[i];
                  break;
                }
              }
              document.getElementById("tls").innerHTML = tlsVersion;
              document.getElementById("http").innerHTML = httpVersion;
              document.getElementById("sni").innerHTML = sniValue;
              document.getElementById("ip").innerHTML = ipValue;
              document.getElementById("useragent").innerHTML = userAgent;

            }
          })
        }
        $(document).ready(function () {
          getCDNinfo();
        });
      </script>


      <p style="text-align: center;">Open Source changes everything!</p>

      <p><img src="./assets/svg/ing.svg"></p>
      <p>
        <a href="https://pages.cloudflare.com" target="_blank" rel="noopener noreferrer"><img
            src="./assets/svg/framework.svg"></a>
        <a href="https://www.cloudflare.com/zh-cn/application-services/products/cdn/" target="_blank"
          rel="noopener noreferrer"><img src="./assets/svg/cdn.svg"></a>
        <a href="https://vuejs.org" target="_blank" rel="noopener noreferrer"><img src="./assets/svg/web.svg"></a>
      </p>


      <p style="text-align: center;">
        <script>
          document.write("本次加载耗时: " +
            (performance.getEntriesByType('navigation').reduce((a, b) => a + b.responseEnd - b.startTime, 0) +
              performance.getEntriesByType('resource').reduce((a, b) => a + b.responseEnd - b.startTime, 0)).toFixed(0) +
            "ms")
        </script>
        <p style="text-align: center;">当前 SNI 状态： <span id="sni">正在统计！或可能被浏览器防追踪拦截！</span></p>
        <p style="text-align: center;">当前 TLS 版本： <span id="tls">正在统计！或可能被浏览器防追踪拦截！</span></p>
        <p style="text-align: center;">当前 HTTP 版本： <span id="http">正在统计！或可能被浏览器防追踪拦截！</span></p>
        <p style="text-align: center;">当前您的客户端 IP 是： <span id="ip">正在统计！或可能被浏览器防追踪拦截！</span></p>
        <p style="text-align: center;">当前分配的 CDN 节点是: <span id="cdn">正在统计！或可能被浏览器防追踪拦截！</span></p>
        <p style="text-align: center;">您的 UserAgent 信息是: <span id="useragent">正在统计！或可能被浏览器防追踪拦截！</span></p>

      </p>




    </footer>
  </section>
  <div class="z-[999999999]">
    <Toaster />
  </div>
  <AlertDialog :open="authStatus">
    <AlertDialogContent>
      <AlertDialogHeader>
        <AlertDialogTitle>请输入登录密码</AlertDialogTitle>
        <AlertDialogDescription>
        </AlertDialogDescription>
      </AlertDialogHeader>
      <Input type="text" placeholder="Password" v-model="loginPassword" />
      <AlertDialogFooter>
        <Button :disabled="loginStatus" @click="loginFn">
          <Loader2 v-show="loginStatus" class="w-4 h-4 mr-2 animate-spin" />登录
        </Button>
      </AlertDialogFooter>
    </AlertDialogContent>
  </AlertDialog>
</template>


<script setup lang="ts">
import { ref, markRaw, onMounted } from 'vue'
import * as echarts from "echarts";
import { Button } from '@/components/ui/button'
import { Loader2 } from 'lucide-vue-next'
import { Skeleton } from '@/components/ui/skeleton'
import { ScrollArea } from '@/components/ui/scroll-area'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Select, SelectContent, SelectGroup, SelectItem, SelectLabel, SelectTrigger, SelectValue, } from '@/components/ui/select'
import { Alert, AlertDescription } from '@/components/ui/alert'
import vh from 'vh-plugin'
import { Toaster } from '@/components/ui/toast'
import { useToast } from '@/components/ui/toast/use-toast'
const { toast } = useToast();
// 弹窗
import { AlertDialog, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, } from '@/components/ui/alert-dialog'
import { Input } from '@/components/ui/input'

// 登录
const authStatus = ref<boolean>(false)
const session = ref<string>(localStorage.getItem('session') || '')
const loginStatus = ref<boolean>(false)
const loginPassword = ref<string>('')
const loginFn = async () => {
  if (!loginPassword.value) return toast({ description: '请输入密码', variant: 'destructive' });
  loginStatus.value = true;
  const res = await fetch('/api', { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify({ type: 'Login', session: loginPassword.value }) })
  await new Promise(resolve => setTimeout(resolve, 666))
  loginStatus.value = false;
  const data = await res.json()
  if (!data.success) return toast({ description: data.message, variant: 'destructive' });
  localStorage.setItem('session', loginPassword.value)
  session.value = loginPassword.value
  authStatus.value = false;
  // 站点列表
  getSiteList()
}

// 站点列表
const siteList = ref<Array<string>>([])
const siteValue = ref<string>('')
const timeList = [{ name: 'Today', value: 'today' }, { name: 'Yesterday', value: '1d' }, { name: 'Last 7 days', value: '7d' }, { name: 'Last 30 days', value: '30d' }, { name: 'Last 60 days', value: '60d' }, { name: 'Last 90 days', value: '90d' }]
const timeValue = ref<string>('today')
const getSiteList = async () => {
  vh.showLoading()
  try {
    const res = await fetch('/api', { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify({ type: 'list', session: session.value }) })
    const data = await res.json()
    if (data.code && data.code === 401) {
      localStorage.clear()
      authStatus.value = true
    }
    if (!data.success) return toast({ description: data.message, variant: 'destructive' });
    siteList.value = data.data;
    siteValue.value = siteList.value[0]
    if (siteValue.value) getDatas()
  } catch (error) {
    console.log(error);
  } finally {
    vh.hideLoading()
  }
}

// 获取数据
const resData = ref<any>({ visit: {} })
const tempResData = ref<any>({ visit: {} })
const getDatasStatus = ref<boolean>(false)
const getDatas = async () => {
  // 清空数据
  resData.value = { visit: {} }
  tempResData.value = { visit: {} }
  // 获取数据
  const pmsARR = ['visit', 'path', 'referrer', 'os', 'soft', 'area', 'echarts'];
  getDatasStatus.value = true
  vh.showLoading()
  const promisesForEach: Array<Promise<any>> = [];
  pmsARR.forEach((i: any) => {
    const p = new Promise((r) => {
      (async () => {
        try {
          const pms = { type: i, siteID: siteValue.value, time: timeValue.value, session: session.value }
          const res = await fetch('/api', { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify(pms) })
          const data = await res.json()
          if (data.code && data.code === 401) {
            localStorage.clear()
            authStatus.value = true;
          }
          if (!data.success) return toast({ description: data.message, variant: 'destructive' });
          tempResData.value[i] = i == 'echarts' ? renderEcharts(data.data.map((i: any) => `${i.name}${['today', '1d'].includes(timeValue.value) ? '点' : '日'}`), data.data.map((i: any) => `${i.value}`)) : data.data
        } catch (error) {
          console.log(error);
        } finally {
          // Promise执行完毕触发
          r(true);
        }
      })();
    });
    promisesForEach.push(p);
  })
  await Promise.all(promisesForEach);
  getDatasStatus.value = false;
  vh.hideLoading()
  // 渲染数据
  resData.value = { ...tempResData.value }
}

// 获取ICON
const getIconUrl = (url: string) => {
  if (!url) return 'https://icons.duckduckgo.com/ip3/none.ico'
  const _url = new URL(url)
  return `https://icons.duckduckgo.com/ip3/${_url.hostname}.ico`
}

// 获取Area ICON
const getIcon = (code: string) => `${location.origin}/icon/${code}.png`

// 渲染图表
const echartsDOM = ref<HTMLCanvasElement>();
const canvasMain = ref<any>();
const renderEcharts = async (dateList: Array<any>, valueList: Array<any>) => {
  const option = {
    grid: { left: "0", right: "0", bottom: "0", top: "10", containLabel: true },
    xAxis: {
      type: "category",
      data: dateList,
      axisLabel: { color: "#959BAA" },
      axisLine: { lineStyle: { color: "rgba(255,255,255,0.56)", width: 2, type: "dashed" } }
    },
    yAxis: { type: "value", axisLabel: { color: "#959BAA" } },
    tooltip: { trigger: "axis" },
    series: [
      {
        data: valueList,
        type: "line",
        smooth: true,
        emphasis: {
          focus: "series",
          itemStyle: { borderWidth: 2 },
          areaStyle: {
            color: {
              colorStops: [
                { offset: 0, color: "#DAE4FF" },
                { offset: 1, color: "#ffffff" }
              ],
              x: 0,
              y: 0,
              x2: 0,
              y2: 1,
              type: "linear",
              global: false
            }
          }
        },
        lineStyle: {
          width: 2,
          color: {
            colorStops: [{ offset: 1, color: "#6F94F1" }],
            x: 0,
            y: 0,
            x2: 1,
            y2: 0,
            type: "linear",
            global: false
          }
        },
        showSymbol: false,
        areaStyle: {
          opacity: 1,
          color: {
            colorStops: [
              { offset: 0, color: "#DAE4FF" },
              { offset: 1, color: "#ffffff" }
            ],
            x: 0,
            y: 0,
            x2: 0,
            y2: 1,
            type: "linear",
            global: false
          }
        }
      }
    ]
  };
  canvasMain.value.setOption(option);
};

onMounted(() => {
  //   图表
  canvasMain.value = markRaw(echarts.init(echartsDOM.value, null, { renderer: "svg", useDirtyRect: true }));
  window.addEventListener("resize", canvasMain.value.resize);
  // 站点列表
  getSiteList()
})
</script>
<style>
.fixed.inset-0.z-50,
.fixed.grid.w-full.max-w-lg.shadow-lg.duration-200 {
  z-index: 99999999;
}
</style>

<style scoped>
@import '@/assets/index.less';
</style>
