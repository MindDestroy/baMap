<template>
  <div class="map">
    <div class="menu">
      <v-card width="256" height="100%" class="mx-auto">
        <v-navigation-drawer permanent>
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title class="title"
                >Belsendi Azamat</v-list-item-title
              >
              <v-list-item-subtitle>карта</v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>

          <v-divider></v-divider>

          <v-list dense nav>
            <v-list-item link>
              <v-list-item-icon>
                <v-icon>mdi-view-dashboard</v-icon>
              </v-list-item-icon>

              <v-list-item-content>
                <v-list-item-title>Районы</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-card class="mx-auto" outlined>
              <v-list-item three-line>
                <v-list-item-content>
                  <div class="overline">Обращения</div>
                  <v-select
                    :items="regions"
                    item-text="name"
                    label="Выберите регион"
                    v-model="regionModel"
                    persistent-hint
                    return-object
                    solo
                    @change="appealChanged"
                  ></v-select>
                  <!-- <v-radio-group v-model="regionGroup">
                    <v-radio
                      v-for="(n, key) in regions"
                      :key="key"
                      :id="n.id"
                      :label="n.name"
                      :value="n.id"
                      @change="appealChanged"
                    ></v-radio>
                  </v-radio-group> -->
                </v-list-item-content>
              </v-list-item>
            </v-card>
            <v-list-item link>
              <v-list-item-icon>
                <v-icon>mdi-image</v-icon>
              </v-list-item-icon>

              <v-list-item-content>
                <v-list-item-title>Статус</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-card class="mx-auto" outlined v-show="items[1].show">
              <v-list-item three-line>
                <v-list-item-content>
                  <div class="overline">Важные</div>
                  <div class="scroll">
                    <v-checkbox
                      v-for="(n, key) in category"
                      :value="n.id"
                      v-model="categoryCheck"
                      :key="key"
                      :label="n.name"
                      :color="n.id == 1 ? '#33b5e5' : 
                      n.id == 2 ? '#0099CC' : 
                      n.id == 3 ? '#00C851' : 
                      n.id == 4 ? '#ffbb33' : 
                      n.id == 5 ? '#aa66cc' : 
                      n.id == 6 ? '#ff4444' : 
                      n.id == 7 ? '#ffbb33' : 
                      n.id == 8 ? '#ffbb33' : 
                      n.id == 9 ? '#ffbb33' :
                      'ff4444'"
                      @change="categoryChanged"
                    ></v-checkbox>
                  </div>
                </v-list-item-content>
              </v-list-item>
            </v-card>
          </v-list>
        </v-navigation-drawer>
      </v-card>
    </div>
    <div id="map" style="width: 100%; height: 100vh"></div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      category: [],
      regions: [{name: 'Muha'}, {name: 'Adik'}],
      items: [
        {
          id: "appeal_all",
          title: "Обращения",
          icon: "mdi-view-dashboard",
          show: true,
        },
        {
          id: "appeal_resolve",
          title: "Категории",
          icon: "mdi-image",
          show: true,
        },
      ],
      appealItem: [{ title: "Все" }, { title: "Решено" }],
      radioGroup: "Все",
      regionModel: '',
      checkbox: "",
      right: null,
      status: 0,
      categoryCheck: [],
    };
  },
  methods: {
    mapAppeal(category, regionMod) {
      var queryArray = [];
      var query = "0";
      if (category != null) {
        category.forEach((element) => {
          queryArray.push(element);
        });
        query = queryArray.join(",");
      }
      window.clusterer.removeAll();
      window.ymap.geoObjects.removeAll();
      var myCollection = [];
      axios
        .get(
          "https://static.belsendiazamat.kz/api/v3/map/getAppeals?adm_divs=" + regionMod.id + "&status=" +
            this.status +
            "&category=" +
            query
        )
        .then((res) => {
          if (res.data.length > 0)
            for (var i = 0, len = res.data.length; i < len; i++) {
              myCollection[i] = new ymaps.Placemark(
                [res.data[i].latitude, res.data[i].longitude],
                {
                  balloonContentHeader: `<font size=3><b><a target="_blank" href="https://static.belsendiazamat.kz/appeals/${res.data[i].id}">${res.data[i].id}</a></b></font>`,
                  balloonContentBody: `<p>${res.data[i].added_visitor.user.first_name} ${res.data[i].added_visitor.user.last_name}</p> <p>Адрес: ${res.data[i].address} </p> <p>${res.data[i].text}</p> <p>${res.data[i].status.name}</p> `,
                  balloonContentFooter: `<font size=1>Дата: </font> <strong>${res.data[i].create_date}</strong>`,
                  clusterCaption:
                    "Обращение <strong>" + res.data[i].id + "</strong>",
                },
                {
                  preset: "islands#violetIcon",
                  iconColor:
                    res.data[i].status.id === 6
                      ? "#ff4444"
                      : res.data[i].status.id === 1
                      ? "#33b5e5"
                      : res.data[i].status.id === 2
                      ? "#0099CC"
                      : res.data[i].status.id === 3
                      ? "#00C851"
                      : res.data[i].status.id === 4
                      ? "#ffbb33"
                      : res.data[i].status.id === 5
                      ? "#aa66cc"
                      : res.data[i].status.id === 6
                      ? "#ff4444"
                      : res.data[i].status.id === 8
                      ? "#ffbb33"
                      : "#ff4444",
                }
              );
            }
          clusterer.add(myCollection);
          window.ymap.geoObjects.add(clusterer);
          clusterer.options.set({
            gridSize: 80,
            clusterDisableClickZoom: true,
          });

          window.ymap.setBounds(clusterer.getBounds(), {
            checkZoomRange: true,
          });
        });
    },
    appealChanged() {
      console.log(this.regionModel.id);
      this.mapAppeal(null, this.regionModel.id)
      // if (this.radioGroup == "Решено") {
      //   this.status = 0;
      //   this.mapAppeal();
      // } else {
      //   this.status = 2;
      //   this.mapAppeal();
      // }
    },
    categoryChanged() {
      this.categoryCheck.length > 0
        ? this.mapAppeal(this.categoryCheck)
        : this.mapAppeal();
    },
  },
  mounted() {
    localStorage.getItem("userlog") != null &&
    localStorage.getItem("userlog") == "true"
      ? null
      : this.$router.push("/");
    axios
      .get("https://static.belsendiazamat.kz/fast_api/get_categories.php")
      .then((result) => {
        this.category = result.data;
      });
      axios.get("https://static.belsendiazamat.kz/api/v3/refBook/getAdmDivisions?has_parent=false").then((result)=> {
        axios.get("https://static.belsendiazamat.kz/api/v3/refBook/getAdmDivisions?parent=1").then((value)=> {
          this.regions = value.data.result.adm_divisions;
          // this.regionModel = result.data.adm_divisions[0]
        });
        
      });
    ymaps.ready(() => {
      let myPlacemark;
      var map = new window.ymaps.Map(
          "map",
          {
            center: [49.807754, 73.088504],
            zoom: 9,
            behaviors: ["default", "scrollZoom"],
          },
          {
            searchControlProvider: "yandex#search",
          }
        ),
        clusterer = new ymaps.Clusterer({
          preset: "islands#",
          groupByCoordinates: false,
          clusterDisableClickZoom: true,
          clusterHideIconOnBalloonOpen: false,
          geoObjectHideIconOnBalloonOpen: false,
        }),
        geoObjects = [];

      var myCollection = [];
      window.ymap = map;
      window.myCollection = myCollection;
      axios
        .get("https://static.belsendiazamat.kz/fast_api/get_appeals.php")
        .then((res) => {
          for (var i = 0, len = res.data.length; i < len; i++) {
            myCollection[i] = new ymaps.Placemark(
              [res.data[i].latitude, res.data[i].longitude],
              {
                balloonContentHeader: `<font size=3><b><a target="_blank" href="https://static.belsendiazamat.kz/appeals/${res.data[i].id}">${res.data[i].id}</a></b></font>`,
                balloonContentBody: `<p>${res.data[i].added_visitor.user.first_name} ${res.data[i].added_visitor.user.last_name}</p> <p>Адрес: ${res.data[i].address} </p><p>${res.data[i].text}</p> <p>${res.data[i].status.name}</p>`,
                balloonContentFooter: `<font size=1>Дата: </font> <strong>${res.data[i].create_date}</strong>`,
                clusterCaption:
                  "Обращение <strong>" + res.data[i].id + "</strong>",
              },
              {
                preset: "islands#",
                iconColor:
                    res.data[i].status.id === 6
                      ? "#ff4444"
                      : res.data[i].status.id === 1
                      ? "#33b5e5"
                      : res.data[i].status.id === 2
                      ? "#0099CC"
                      : res.data[i].status.id === 3
                      ? "#00C851"
                      : res.data[i].status.id === 4
                      ? "#ffbb33"
                      : res.data[i].status.id === 5
                      ? "#aa66cc"
                      : res.data[i].status.id === 6
                      ? "#ff4444"
                      : res.data[i].status.id === 8
                      ? "#ffbb33"
                      : "#ff4444",
              }
            );
          }
          clusterer.add(myCollection);
          map.geoObjects.add(clusterer);
          clusterer.options.set({
            gridSize: 80,
            clusterDisableClickZoom: true,
          });

          map.setBounds(clusterer.getBounds(), {
            checkZoomRange: true,
          });
          window.clusterer = clusterer;
        });
    });
  },
};
</script>

<style lang="less">
.map {
  display: flex;
}
.scroll {
  max-height: 80vh;
  overflow: scroll;
  label {
    font-size: 12px;
  }
}
</style>