<template>
  <q-layout view="hHh lpR fff">
    <q-header>
      <q-toolbar class="title-toolbar">
        <q-toolbar-title> </q-toolbar-title>
        <div>
          <q-btn flat @click="scrollToSection('todayWeather')">
            <img
              :src="require('assets/icon/天氣圖卡.png')"
              class="header-icon"
            />
          </q-btn>
          <q-btn
            flat
            v-if="weatherWarning"
            @click="scrollToSection('weatherWarning')"
          >
            <img
              :src="require('assets/icon/天氣特報.png')"
              class="header-icon"
            />
          </q-btn>
          <q-btn flat @click="scrollToSection('moreInfo')">
            <img
              :src="require('assets/icon/更多資訊.png')"
              class="header-icon"
            />
          </q-btn>
          <router-link to="/index.html" class="q-btn q-btn--flat">
            <img
              :src="require('assets/icon/目前資訊.png')"
              class="header-icon"
            />
          </router-link>
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container id="todayWeather" style="margin: 0 auto">
      <div class="q-pa-md">
        <!-- 今日天氣和未來天氣 -->
        <div class="top-title">
          <div class="weather-icon-column" v-if="weatherView === 'today'">
            <q-img :src="require('assets/icon/bear.png')" class="icon" />
            <span class="icon-text"><strong>天氣熊好懂</strong></span>
          </div>
          <div class="weather-icon-column" v-if="weatherView === 'future'">
            <q-img :src="require('assets/icon/AI.png')" class="icon" />
            <span class="icon-text"><strong>AI智繪月曆</strong></span>
          </div>
        </div>
        <!-- 選擇地區 -->
        <div class="frame-container">
          <div class="top-banner">
            <q-img
              :src="require('assets/icon/pin.png')"
              class="small-icon q-mr-md q-ml-sm"
              v-if="weatherView === 'today'"
            />
            <div
              class="location-select-container"
              v-if="weatherView === 'today'"
            >
              <q-select
                v-model="selectedCity"
                :options="cityOptions"
                label="選擇縣市"
                @update:model-value="onCityChange"
                dense
                outlined
                class="location-select"
              />
              <q-select
                v-model="selectedDistrict"
                :options="districtOptions"
                label="選擇鄉鎮"
                @update:model-value="updateWeatherImage"
                dense
                outlined
                class="location-select"
              />
            </div>
            <q-img
              :src="require('assets/icon/bear.png')"
              class="icon q-mr-md q-ml-sm"
              v-if="weatherView === 'future'"
            />
            <span v-if="weatherView === 'future'" class="icon-text">
              <strong>想知道未來一個月的天氣類型是什麼嗎?</strong>
            </span>
            <!-- 天氣icon說明 -->
            <q-btn
              flat
              icon="info"
              @click="showFutureInfoDialog"
              label="呼叫熊好懂"
              class="q-ml-xs"
              v-if="weatherView === 'future'"
            />
            <q-dialog v-model="futureInfoDialog" class="custom-dialog">
              <q-card class="custom-card">
                <q-tabs v-model="tab" class="text-teal" inline-label dense>
                  <q-tab
                    v-for="(info, index) in infoData"
                    :key="index"
                    :name="info.label"
                  >
                    <div class="tab-content">
                      <q-img :src="info.icon" class="tab-icon" />
                      <div>{{ info.label }}</div>
                    </div>
                  </q-tab>
                </q-tabs>
                <q-tab-panels v-model="tab" animated>
                  <q-tab-panel
                    v-for="(info, index) in infoData"
                    :key="index"
                    :name="info.label"
                  >
                    <q-card-section>
                      <div v-html="info.content"></div>
                    </q-card-section>
                  </q-tab-panel>
                </q-tab-panels>
                <q-card-actions align="right">
                  <q-btn flat label="關閉" v-close-popup />
                </q-card-actions>
              </q-card>
            </q-dialog>
            <div class="button-container">
              <!-- 下載按鈕 -->
              <q-btn
                v-if="weatherView === 'today' && !downloadingTodayWeather"
                round
                icon="download"
                @click="downloadTodayWeatherImage"
                class="download-button"
              />
              <q-spinner-ios
                v-else-if="weatherView === 'today'"
                color="primary"
                size="3em"
              />
              <q-btn
                v-if="weatherView === 'future' && !downloadingFutureWeather"
                round
                icon="download"
                @click="downloadFutureWeatherImage"
                class="download-button"
              />
              <q-spinner-ios
                v-else-if="weatherView === 'future'"
                color="primary"
                size="3em"
              />
              <!-- 切換圖卡按鈕 -->
              <q-btn
                class="top-banner-button"
                :class="{ active: weatherView === 'today' }"
                @click="showTodayWeather"
              >
                <strong>今日天氣</strong>
              </q-btn>
              <q-btn
                class="top-banner-button"
                :class="{ active: weatherView === 'future' }"
                @click="showFutureWeather"
              >
                <strong>未來天氣</strong>
              </q-btn>
            </div>
          </div>
          <div v-if="weatherWarning && weatherView === 'today'" class="marquee">
            <div class="marquee-content">
              <q-img
                :src="require('assets/icon/warning.png')"
                style="
                  width: 30px;
                  height: 30px;
                  margin-right: 10px;
                  margin-left: 40px;
                  margin-bottom: 5px;
                "
              />
              <span>{{ weatherWarning }}</span>
              <q-img
                :src="require('assets/icon/warning.png')"
                style="
                  width: 30px;
                  height: 30px;
                  margin-right: 10px;
                  margin-left: 60px;
                  margin-bottom: 5px;
                "
              />
              <span style="color: black"
                >最新消息以氣象署為主，請隨時關注氣象署公布資訊~</span
              >
            </div>
          </div>
          <div class="iframe-wrapper" v-if="weatherView === 'today'">
            <q-img :src="weatherImgSrc" />
          </div>
          <div class="calendar-wrapper" v-if="weatherView === 'future'">
            <FutureWeatherCalendar />
          </div>
        </div>
      </div>

      <q-separator
        class="q-itemider"
        size="70px"
        color="transparent"
        id="weatherWarning"
      />

      <!-- 天氣特報 -->
      <div class="q-pa-md" v-if="weatherWarning">
        <div class="top-title">
          <div class="weather-icon-column">
            <q-img :src="require('assets/icon/clear-sky.png')" class="icon" />
            <span class="icon-text">
              <strong>天氣特報</strong>
            </span>
            <!-- 說明對話框 -->
            <q-btn
              flat
              icon="help_outline"
              @click="showInfoDialogWarning"
              label="這是什麼 ?"
              class="q-ml-xs"
            />
            <q-dialog v-model="infoDialog">
              <q-card>
                <q-card-section>
                  <div class="text-h6"><strong>圖資說明</strong></div>
                </q-card-section>
                <q-card-section class="q-pt-none">
                  <div v-html="currentInfo"></div>
                </q-card-section>
                <q-card-actions align="right">
                  <q-btn flat label="關閉" v-close-popup />
                </q-card-actions>
              </q-card>
            </q-dialog>
          </div>
        </div>

        <div class="frame-container">
          <div class="top-banner">
            <q-img :src="iconSrc" class="icon" />
            <span class="icon-text"
              ><strong>{{ selectedWarnLabel }}</strong></span
            >
            <q-btn
              flat
              icon="info"
              @click="showWarnInfoDialog"
              label="呼叫熊好懂 !"
              class="q-ml-xs"
            />
            <q-dialog v-model="warnInfoDialog">
              <q-card>
                <q-card-section>
                  <div v-html="warnInfoTitle" class="text-h6"></div>
                </q-card-section>
                <q-card-section class="q-pt-none">
                  <div v-html="warnInfoContent"></div>
                </q-card-section>
                <q-card-actions align="right">
                  <q-btn flat label="關閉" v-close-popup />
                </q-card-actions>
              </q-card>
            </q-dialog>
            <div class="button-container">
              <q-btn
                v-for="warnbtn in warnbuttons"
                :key="warnbtn.url"
                class="top-banner-button"
                :class="{ active: iframeWarnSrc === warnbtn.url }"
                @click="
                  changeIframeWarnSrc(warnbtn.url, warnbtn.label, warnbtn.icon)
                "
              >
                <strong>{{ warnbtn.label }}</strong>
              </q-btn>
            </div>
          </div>
          <div class="iframe-wrapper">
            <iframe
              :src="iframeWarnSrc"
              scrolling="no"
              class="weather-iframe"
              :style="{ height: warnHeight, width: '1160px', border: 'none' }"
            ></iframe>
          </div>
        </div>
      </div>

      <q-separator
        class="q-itemider"
        size="70px"
        color="transparent"
        id="moreInfo"
      />

      <!-- 日常開關圖資 -->
      <div class="q-pa-md">
        <!-- 開關 -->
        <div class="top-title">
          <div class="weather-icon-column">
            <q-img :src="require('assets/icon/info.png')" class="icon" />
            <span class="icon-text"><strong>看看更多資訊</strong></span>
            <!-- 說明對話框 -->
            <q-btn
              flat
              icon="help_outline"
              @click="showInfoDialogDaily"
              label="這是什麼 ?"
              class="q-ml-xs"
            />
            <q-dialog v-model="infoDialog">
              <q-card>
                <q-card-section>
                  <div class="text-h6"><strong>圖資說明</strong></div>
                </q-card-section>
                <q-card-section class="q-pt-none">
                  <div v-html="currentInfo"></div>
                </q-card-section>
                <q-card-actions align="right">
                  <q-btn flat label="關閉" v-close-popup />
                </q-card-actions>
              </q-card>
            </q-dialog>
            <div class="menu-container q-mr-md">
              <q-btn flat icon="menu">
                <span class="q-ml-sm" style="font-size: 17px"
                  ><strong>點我看看 !</strong></span
                >
                <q-menu>
                  <div class="row no-wrap q-pl-md q-pr-xl q-pt-md q-pb-md">
                    <div class="column">
                      <div class="q-mb-md" style="font-size: 20px">
                        <q-img
                          :src="require('assets/icon/bear.png')"
                          class="icon"
                        /><strong>試試看開關 !</strong>
                      </div>
                      <q-item
                        v-for="dailybtn in dailybuttons"
                        :key="dailybtn.url"
                      >
                        <q-item-section avatar>
                          <q-toggle
                            v-model="dailybtn.active"
                            @update:model-value="onToggleChange(dailybtn)"
                          />
                        </q-item-section>
                        <q-item-section>
                          <q-item-label style="font-size: 17px">{{
                            dailybtn.caption
                          }}</q-item-label>
                          <q-item-label caption>{{
                            dailybtn.title
                          }}</q-item-label>
                        </q-item-section>
                      </q-item>
                    </div>
                  </div>
                </q-menu>
              </q-btn>
            </div>
          </div>
          <div
            v-if="activeDailySrcs.length === 0"
            style="max-width: 1140px; padding-top: 20px"
          >
            <div class="weather-icon-column">
              <q-img :src="require('assets/icon/sad-bear.png')" class="icon" />
              <span style="font-size: 17px">什麼都還沒開哦...</span>
            </div>
          </div>
        </div>
        <!-- 圖資 -->

        <div v-for="daily in activeDailySrcs" :key="daily.url">
          <div class="frame-container">
            <div class="top-banner">
              <q-img :src="require(`assets/icon/${daily.icon}`)" class="icon" />
              <span class="icon-text"
                ><strong>{{ daily.label }}</strong></span
              >
              <q-btn
                flat
                icon="info"
                @click="showDailyInfoDialog(daily)"
                label="呼叫熊好懂 !"
                class="q-ml-xs"
              />
              <q-dialog v-model="dailyInfoDialog">
                <q-card>
                  <q-card-section>
                    <div v-html="dailyInfoTitle" class="text-h6"></div>
                  </q-card-section>
                  <q-card-section class="q-pt-none">
                    <div v-html="dailyInfoContent"></div>
                  </q-card-section>
                  <q-card-actions align="right">
                    <q-btn flat label="關閉" v-close-popup />
                  </q-card-actions>
                </q-card>
              </q-dialog>
            </div>
            <div class="iframe-wrapper">
              <iframe
                scrolling="no"
                class="weather-iframe"
                :src="daily.url"
                :style="{
                  height: daily.height,
                  width: '1160px',
                  border: 'none',
                }"
              ></iframe>
            </div>
          </div>
          <q-separator class="q-itemider" size="20px" color="transparent" />
        </div>
      </div>
      <q-separator class="q-itemider" size="20px" color="transparent" />
      <q-footer class="bg-grey-8 text-white q-pa-lg">
        <q-toolbar>
          <q-toolbar-title class="footer">
            <span
              >Copyright © NCDR人才培育暨暑期實習2024 - 氣象組<br />地址 :
              新北市新店區北新路三段200號9樓<br
            /></span>
          </q-toolbar-title>
        </q-toolbar>
      </q-footer>
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref, watch } from "vue";
import html2canvas from "html2canvas";
import { warnbuttons } from "src/data/warnData.js";
import { dailybuttons } from "src/data/dailyData.js";
import { cityDistricts } from "src/data/cityData.js";
import { infoData } from "src/assets/calendar/weatherIcons.js";
import FutureWeatherCalendar from "src/components/FutureWeatherCalendar.vue";

export default defineComponent({
  name: "HistoryPage",
  components: {
    FutureWeatherCalendar,
  },
  setup() {
    // 選擇地區
    const cityOptions = cityDistricts.map((city) => city.cityName);
    const selectedCity = ref("新北市");
    const selectedDistrict = ref("新店區");
    const districtOptions = ref([]);

    // 今日天氣和未來天氣
    const weatherView = ref("today");
    const weatherImgSrc = ref("");
    const downloadingTodayWeather = ref(false);
    const downloadingFutureWeather = ref(false);

    const generateTodayWeatherImgSrc = (city, district) => {
      return require(`assets/pics/${city}${district}.png`);
    };

    const showTodayWeather = () => {
      weatherImgSrc.value = generateTodayWeatherImgSrc(
        selectedCity.value,
        selectedDistrict.value
      );
      weatherView.value = "today";
    };

    const showFutureWeather = () => {
      weatherView.value = "future";
    };

    // 初始化選擇地區和圖片
    const initializeLocationAndImage = () => {
      onCityChange(selectedCity.value);
      showTodayWeather();
    };

    // 天氣特報WATCH api
    const iframeWarnSrc = ref("https://watch.ncdr.nat.gov.tw/w_68");
    const selectedWarnLabel = ref("高溫資訊燈號");
    const iconSrc = ref(require("assets/icon/hot.png"));
    const warnHeight = ref("760px");
    const weatherWarning = ref(null);

    const changeIframeWarnSrc = (newSrc, label, icon, height) => {
      iframeWarnSrc.value = newSrc;
      selectedWarnLabel.value = label;
      iconSrc.value = require(`assets/icon/${icon}`);
      warnHeight.value = height || "760px";
    };

    const downloadTodayWeatherImage = async () => {
      downloadingTodayWeather.value = true;
      try {
        const link = document.createElement("a");
        link.href = weatherImgSrc.value;
        link.download = `${selectedCity.value}_${selectedDistrict.value}.png`;
        link.style.display = "none";
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      } finally {
        downloadingTodayWeather.value = false;
      }
    };

    const downloadFutureWeatherImage = async () => {
      downloadingFutureWeather.value = true;
      try {
        const element = document.querySelector(".future-weather-calendar");
        const canvas = await html2canvas(element, {
          useCORS: true,
          scale: 2,
        });
        const link = document.createElement("a");
        link.href = canvas.toDataURL("image/png");
        link.download = "future_calendar.png";
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      } finally {
        downloadingFutureWeather.value = false;
      }
    };

    // 天氣警特報&高溫低溫特報&日常圖資開關
    const fetchWeatherAndTemperatureWarnings = async (
      cityName,
      districtName
    ) => {
      const city = cityDistricts.find((city) => city.cityName === cityName);
      if (!city) {
        console.error(`Invalid city name: ${cityName}`);
        return;
      }

      const areaName = city.areaName;
      const tempJsonUrl = `/data/api/cwa_data_${city.countyCode}.json`;
      const warnJsonUrl = `/data/api/warn_data.json`;
      const airJsonUrl = `/data/api/air_data.json`;

      try {
        const [warnResponse, tempResponse, airResponse] = await Promise.all([
          fetch(warnJsonUrl),
          fetch(tempJsonUrl),
          fetch(airJsonUrl),
        ]);

        if (!warnResponse.ok || !tempResponse.ok || !airResponse.ok) {
          throw new Error("Failed to fetch data JSON files");
        }

        const [warnData, cwaData, airData] = await Promise.all([
          warnResponse.json(),
          tempResponse.json(),
          airResponse.json(),
        ]);

        const cityWarn = warnData.records.location.find(
          (location) => location.locationName === cityName
        );

        let warnings = [];
        let severeRain = false;
        let typhoon = false;

        if (cityWarn) {
          const hazards = cityWarn.hazardConditions.hazards;

          hazards.forEach((hazard) => {
            const phenomena = hazard.info.phenomena;
            const significance = hazard.info.significance;
            const warning = `${phenomena}${significance}`;
            console.log(`City: ${cityName}, Weather Warning: ${warning}`);

            const warningMessage = (phenomena) => {
              switch (phenomena) {
                case "大雨":
                  severeRain = true;
                  return "當前區域未來可能有強降雨發生，記得攜帶雨傘和關注防災資訊！";
                case "豪雨":
                  severeRain = true;
                  return "當前區域未來可能有豪雨發生，記得攜帶雨傘和關注防災資訊！";
                case "大豪雨":
                  severeRain = true;
                  return "當前區域未來可能有大豪雨發生，記得攜帶雨傘和關注防災資訊！";
                case "超大豪雨":
                  severeRain = true;
                  return "當前區域未來可能有超大豪雨發生，記得攜帶雨傘和關注防災資訊！";
                case "海上陸上颱風":
                  typhoon = true;
                  return "當前區域未來可能受到颱風影響，記得盡量避免外出、做好防範措施和關注防災資訊！";
                default:
                  return "";
              }
            };

            if (
              ["大雨", "豪雨", "大豪雨", "超大豪雨", "海上陸上颱風"].includes(
                phenomena
              )
            ) {
              warnings.push(`${warning} : ${warningMessage(phenomena)}`);
            }
          });
        }

        const districtData = cwaData.records.locations[0].location.find(
          (location) => location.locationName === districtName
        );

        if (districtData) {
          const temperatures = districtData.weatherElement.find(
            (element) => element.elementName === "T"
          ).time;

          const comfortIndices = districtData.weatherElement.find(
            (element) => element.elementName === "CI"
          ).time;

          const precipitationProbabilities = districtData.weatherElement.find(
            (element) => element.elementName === "PoP6h"
          ).time;

          if (temperatures.length > 0) {
            const now = new Date();
            const currentHour = now.getHours();
            const currentMinute = now.getMinutes();
            const startHour = determineStartHour(currentHour, currentMinute);

            const relevantTemperature = temperatures.find(
              (temp) => new Date(temp.dataTime).getHours() === startHour
            );

            const relevantComfortIndex = comfortIndices.find(
              (ci) => new Date(ci.dataTime).getHours() === startHour
            );

            const relevantPrecipitationProbabilityIndex =
              precipitationProbabilities.findIndex((pop) => {
                const startTime = new Date(pop.startTime);
                const endTime = new Date(pop.endTime);
                return (
                  startHour >= startTime.getHours() &&
                  startHour < endTime.getHours()
                );
              });

            const relevantPrecipitationProbability =
              precipitationProbabilities[relevantPrecipitationProbabilityIndex];
            const nextPrecipitationProbability =
              precipitationProbabilities[
                relevantPrecipitationProbabilityIndex + 1
              ];

            const allPrecipitationBelowThreshold =
              relevantPrecipitationProbability &&
              nextPrecipitationProbability &&
              parseInt(relevantPrecipitationProbability.elementValue[0].value) <
                40 &&
              parseInt(nextPrecipitationProbability.elementValue[0].value) < 40;

            const cwbButton = dailybuttons.find(
              (btn) => btn.label === "落雨小幫手"
            );
            if (cwbButton) {
              cwbButton.active = !allPrecipitationBelowThreshold;
            }

            if (relevantTemperature) {
              const time = relevantTemperature.dataTime;
              const temp = parseInt(relevantTemperature.elementValue[0].value);
              const comfortIndexText = relevantComfortIndex
                ? relevantComfortIndex.elementValue[1].value
                : "未知";
              const precipitationProbability = relevantPrecipitationProbability
                ? relevantPrecipitationProbability.elementValue[0].value
                : "未知";

              const aqiValue =
                airData.records.find((record) => record.area === areaName)
                  ?.aqi || "未知";

              console.log(
                `City: ${cityName}, District: ${districtName}, Time: ${time}, Temperature: ${temp}, Comfort Index: ${comfortIndexText}, Precipitation Probability: ${precipitationProbability}, AQI: ${aqiValue}`
              );

              // 設定高溫和低溫特報燈號的預設按鈕
              if (temp > 35) {
                changeIframeWarnSrc(
                  warnbuttons.find((btn) => btn.label === "高溫資訊燈號").url,
                  "高溫資訊燈號",
                  "hot.png",
                  "760px"
                );
                warnings.push(
                  "高溫資訊 : 當前區域未來溫度預計超過36度，記得注意防曬與補充水分！"
                );
              } else if (temp < 11) {
                changeIframeWarnSrc(
                  warnbuttons.find((btn) => btn.label === "低溫特報燈號").url,
                  "低溫特報燈號",
                  "cold.png",
                  "760px"
                );
                warnings.push(
                  "低溫特報 : 當前區域未來溫度預計低於10度，記得攜帶外套並注意保暖！"
                );
              }

              if (comfortIndexText !== "未知") {
                const groundStationButton = dailybuttons.find(
                  (btn) => btn.label === "地面測站氣溫監測"
                );
                if (groundStationButton) {
                  groundStationButton.active = !["舒適", "涼爽"].includes(
                    comfortIndexText
                  );
                }
              }

              activeDailySrcs.value = dailybuttons
                .filter((btn) => btn.active)
                .map((btn) => ({
                  url: btn.url,
                  label: btn.label,
                  icon: btn.icon,
                  height: btn.height,
                  title: btn.title,
                  caption: btn.caption,
                  discription: btn.discription,
                  content: btn.content,
                }));
            }

            const aqiButton = dailybuttons.find(
              (btn) => btn.label === "環境部空氣品質AQI預報"
            );
            if (
              aqiButton &&
              airData.records.find((record) => record.area === areaName)
            ) {
              const aqiValue = parseInt(
                airData.records.find((record) => record.area === areaName).aqi
              );
              aqiButton.active = !(aqiValue <= 100);
            }
          }
        }

        // 設定豪雨或颱風的預設按鈕
        if (severeRain) {
          changeIframeWarnSrc(
            warnbuttons.find((btn) => btn.label === "全台示警燈號").url,
            "全台示警燈號",
            "alarm.png",
            "760px"
          );
        } else if (typhoon) {
          changeIframeWarnSrc(
            warnbuttons.find((btn) => btn.label === "颱風路徑").url,
            "颱風路徑",
            "typhoon.png",
            "760px"
          );
        }

        weatherWarning.value = warnings.length > 0 ? warnings.join("｜") : null;
      } catch (error) {
        weatherWarning.value = null;
        console.error("Error fetching weather and temperature data:", error);
      }
    };

    const determineStartHour = (hour, minute) => {
      if (
        (hour == 1 && minute >= 30) ||
        (2 <= hour && hour < 4) ||
        (hour == 4 && minute < 30)
      ) {
        return 3;
      } else if (
        (hour == 4 && minute >= 30) ||
        (5 <= hour && hour < 7) ||
        (hour == 7 && minute < 30)
      ) {
        return 6;
      } else if (
        (hour == 7 && minute >= 30) ||
        (8 <= hour && hour < 10) ||
        (hour == 10 && minute < 30)
      ) {
        return 9;
      } else if (
        (hour == 10 && minute >= 30) ||
        (11 <= hour && hour < 13) ||
        (hour == 13 && minute < 30)
      ) {
        return 12;
      } else if (
        (hour == 13 && minute >= 30) ||
        (14 <= hour && hour < 16) ||
        (hour == 16 && minute < 30)
      ) {
        return 15;
      } else if (
        (hour == 16 && minute >= 30) ||
        (17 <= hour && hour < 19) ||
        (hour == 19 && minute < 30)
      ) {
        return 18;
      } else if (
        (hour == 19 && minute >= 30) ||
        (20 <= hour && hour < 22) ||
        (hour == 22 && minute < 30)
      ) {
        return 21;
      } else {
        return 0; // 00:00
      }
    };

    // 獲取新圖片
    const onCityChange = (cityName) => {
      const city = cityDistricts.find((city) => city.cityName === cityName);
      if (city) {
        districtOptions.value = city.districts.map(
          (district) => district.districtName
        );
        selectedDistrict.value = districtOptions.value.length
          ? districtOptions.value[0]
          : null;
        fetchWeatherAndTemperatureWarnings(cityName, selectedDistrict.value);
      }
    };

    const updateWeatherImage = () => {
      weatherImgSrc.value = generateTodayWeatherImgSrc(
        selectedCity.value,
        selectedDistrict.value
      );
    };

    watch(
      [selectedCity, selectedDistrict],
      ([newCity, newDistrict]) => {
        console.log("Selected City and District:", newCity, newDistrict);
        fetchWeatherAndTemperatureWarnings(newCity, newDistrict);
        updateWeatherImage();
      },
      { immediate: true }
    );

    // 日常開關圖資
    const activeDailySrcs = ref([]);

    dailybuttons.forEach((dailybtn) => {
      if (dailybtn.active) {
        activeDailySrcs.value.push({
          url: dailybtn.url,
          label: dailybtn.label,
          icon: dailybtn.icon,
          height: dailybtn.height,
          title: dailybtn.title,
          caption: dailybtn.caption,
          discription: dailybtn.discription,
          content: dailybtn.content,
        });
      }
    });

    const onToggleChange = (dailybtn) => {
      if (dailybtn.active) {
        activeDailySrcs.value.push({
          url: dailybtn.url,
          label: dailybtn.label,
          icon: dailybtn.icon,
          height: dailybtn.height,
          title: dailybtn.title,
          caption: dailybtn.caption,
          discription: dailybtn.discription,
          content: dailybtn.content,
        });
      } else {
        activeDailySrcs.value = activeDailySrcs.value.filter(
          (daily) => daily.url !== dailybtn.url
        );
      }
    };

    // 快捷路徑按鈕設定
    const scrollToSection = (section) => {
      const element = document.getElementById(section);
      if (element) {
        element.scrollIntoView({ behavior: "smooth" });
      }
    };

    // 顯示信息對話框
    const infoDialog = ref(false);
    const currentInfo = ref("");

    const showInfoDialogWarning = () => {
      currentInfo.value = `
        <p>當有颱風、強降雨、高溫、低溫等天氣現象發生時，</p>
        <p>天氣特報會呈現相關圖資，</p>
        <p>大家要多加注意哦！</p>
        <p>如果對圖資有疑問，不知道該怎麼看的話，</p>
        <p>可以點擊「呼叫熊好懂」查看喔！</p>
      `;
      infoDialog.value = true;
    };

    const showInfoDialogDaily = () => {
      currentInfo.value = `
        <p>這裡的圖資會提供日常所需的氣象資訊，</p>
        <p>右邊的開關設計會根據當下天氣改變，</p>
        <p>留下需要注意的圖資 ~</p>
        <p>如果對圖資有疑問，不知道該怎麼看的話，</p>
        <p>可以點擊「呼叫熊好懂」查看喔！</p>
      `;
      infoDialog.value = true;
    };

    // 圖資說明
    const warnInfoDialog = ref(false);
    const warnInfoTitle = ref("");
    const warnInfoContent = ref("");

    const showWarnInfoDialog = () => {
      const selectedWarn = warnbuttons.find(
        (warnbtn) => warnbtn.label === selectedWarnLabel.value
      );
      if (selectedWarn) {
        warnInfoTitle.value = selectedWarn.discription;
        warnInfoContent.value = selectedWarn.content;
      } else {
        warnInfoTitle.value = "資訊";
        warnInfoContent.value = "無相關資訊";
      }
      warnInfoDialog.value = true;
    };

    // 日常圖資說明
    const dailyInfoDialog = ref(false);
    const dailyInfoTitle = ref("");
    const dailyInfoContent = ref("");

    const showDailyInfoDialog = (daily) => {
      dailyInfoTitle.value = daily.discription;
      dailyInfoContent.value = daily.content;
      dailyInfoDialog.value = true;
    };

    // 月曆說明
    const futureInfoDialog = ref(false);
    const tab = ref(infoData[0].label);
    // 將 infoData 引入 setup 方法
    const infoDataRef = ref(infoData);

    const showFutureInfoDialog = () => {
      futureInfoDialog.value = true;
    };

    initializeLocationAndImage();

    return {
      weatherView,
      weatherImgSrc,
      generateTodayWeatherImgSrc,
      showTodayWeather,
      showFutureWeather,
      dailybuttons,
      warnbuttons,
      iframeWarnSrc,
      selectedWarnLabel,
      iconSrc,
      warnHeight,
      changeIframeWarnSrc,
      cityOptions,
      selectedCity,
      selectedDistrict,
      districtOptions,
      onCityChange,
      updateWeatherImage,
      activeDailySrcs,
      onToggleChange,
      weatherWarning,
      downloadTodayWeatherImage,
      downloadFutureWeatherImage,
      downloadingTodayWeather,
      downloadingFutureWeather,
      scrollToSection,
      infoDialog,
      currentInfo,
      showInfoDialogWarning,
      showInfoDialogDaily,
      warnInfoDialog,
      warnInfoTitle,
      warnInfoContent,
      showWarnInfoDialog,
      dailyInfoDialog,
      dailyInfoTitle,
      dailyInfoContent,
      showDailyInfoDialog,
      showFutureInfoDialog,
      futureInfoDialog,
      tab,
      infoData: infoDataRef,
    };
  },
});
</script>

<style scoped>
.q-header {
  background-color: transparent;
  background-position: center;
  background-repeat: no-repeat;
  display: flex;
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
  width: 100%;
}

.title-toolbar {
  background-color: transparent;
  background-image: url("assets/header.png");
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  max-width: 1250px;
  width: 100%;
  margin: 0 auto; /* 置中 */
  display: flex;
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
}

.header-icon {
  width: 65px;
  height: 65px;
}

.top-title {
  width: 1200px;
  margin: 0 auto;
  padding-bottom: 20px;
}
.weather-icon-column {
  display: flex;
  align-items: center;
  background-color: transparent;
  font-size: 1.5rem;
}

.icon {
  width: 40px;
  height: 40px;
  overflow: hidden;
  margin-right: 10px;
  margin-left: 10px;
}

.small-icon {
  width: 20px;
  height: 20px;
  overflow: hidden;
}

.icon-text {
  font-size: 20px;
}

.frame-container {
  position: relative;
  border: 1px solid #ddd;
  width: 1200px;
  margin: 0 auto;
  box-shadow: 0 5px 8px rgba(0, 0, 0, 0.1);
}

.top-banner {
  display: flex;
  align-items: center;
  background-color: #f0f0f0;
  padding: 5px;
  border-bottom: 1px solid #ddd;
}

.location-select-container {
  display: flex;
  align-items: center;
  margin-right: auto;
}

.location-select {
  margin-right: 10px;
  width: 200px;
}

.button-container {
  display: flex;
  margin-left: auto;
}

.download-button {
  margin-right: 5px;
  background-color: transparent;
}

/* 月曆icon說明 */
.custom-card {
  padding-top: 5px;
  min-width: 680px;
  height: 320px;
  overflow: hidden;
}

.tab-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-weight: 700;
}

.tab-icon {
  width: 40px;
  height: 40px;
  margin-bottom: 6px;
}

.top-banner-button {
  margin: 2px;
  font-size: 15px;
  background-color: transparent;
}

.top-banner-button.active {
  background-color: #b0b0b0;
}

.iframe-wrapper {
  width: 1200px;
  padding: 15px;
  position: relative;
}

.weather-iframe {
  width: 1200px;
  border: none;
  position: relative;
}

.menu-container {
  display: flex;
  align-items: center;
  margin-left: auto;
}

.weather-img {
  width: 100%;
}

.warning-box {
  position: absolute;
  top: 140px;
  right: 630px;
  background-color: rgb(253, 203, 21);
  color: black;
  padding: 10px;
  border-radius: 5px;
  font-size: 20px;
}

.marquee {
  background-color: rgba(255, 228, 94, 0.806);
  color: rgb(222, 11, 11);
  padding: 10px;
  font-size: 1.2rem;
  white-space: nowrap;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  height: 40px;
  margin: 0 auto;
}

.marquee-content {
  display: inline-block;
  padding-top: 5px;
  padding-left: 100%;
  animation: marquee 40s linear infinite;
}

.q-footer {
  font-size: 17px;
  text-align: center;
  width: 1255px;
  right: auto;
}

@keyframes marquee {
  0% {
    transform: translateX(0%);
  }
  100% {
    transform: translateX(-100%);
  }
}
</style>
