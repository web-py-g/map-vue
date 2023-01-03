<template>
  <div class="page">
    <h1>Тестовое задание для Frontend разработчика.</h1>
    <div class="page_points">
      <v-text-field
        class="page__input"
        v-model.trim="pointName"
        v-on:keyup.enter="addNewPoint"
        label="Добавление новой точки"
        hide-details="auto"
        prepend-icon="mdi-map-marker"
        hint="Для ввода используйте Enter"
      ></v-text-field>
      <span class="br"></span>
      <h4>Список точек:</h4>
      <div class="point_list">
        <v-chip-group v-model="selection" column active-class="primary--text">
          <draggable v-model="points" @start="dragStart" @end="dragEnd">
            <v-chip
              draggable
              close
              label
              link
              @click:close="delPoint(point)"
              v-for="point in points"
              :key="point.id"
            >
              {{ point.name }}
            </v-chip>
          </draggable>
        </v-chip-group>
      </div>
    </div>
    <div class="container">
      <div id="map" class="shadow" style="width: 100%; height: 450px"></div>
    </div>
  </div>
</template>

<script>
const ymaps = window.ymaps;
import draggable from "vuedraggable";

export default {
  name: "Map",
  components: {
    draggable,
  },
  data: () => ({
    test: "test",
    controls: ["searchControl"],
    coords: [59.93948689720954, 30.317511196135268],
    points: [],
    markers: [],
    pointName: "",
    myMap: null,
    maxId: 0,
    polyline: null,
    selection: null,
    currentTag: null,
    tags: [
      {
        id: 1,
        name: "Shoping",
      },
      {
        id: 2,
        name: "Art",
      },
      {
        id: 3,
        name: "Tech",
      },
      {
        id: 4,
        name: "Creative Writing",
      },
    ],
  }),
  methods: {
    dragStart() {
      if (this.tags[this.selection])
        this.currentTag = this.tags[this.selection].name;
      else this.currentTag = null;
    },
    dragEnd() {
      var self = this;
      if (this.currentTag) {
        this.tags.forEach((x, i) => {
          if (x.name === self.currentTag) self.selection = i;
        });
      }
      this.initPolyline();
    },
    init() {
      this.myMap = new ymaps.Map("map", {
        center: [59.93948689720954, 30.317511196135268],
        zoom: 11,
        controls: this.controls,
      });
    },
    addNewPoint() {
      if (this.pointName == "") return;
      this.maxId += 1;
      let point = {
        id: this.maxId,
        name: this.maxId + ". " + this.pointName,
      };
      this.pointName = "";

      let center = this.myMap.getCenter();
      let address;
      ymaps.geocode(center).then(function (res) {
        var firstGeoObject = res.geoObjects.get(0);
        address = firstGeoObject.getAddressLine();
        newPlacemark.properties.set({
          hintContent: firstGeoObject.getAddressLine(),
        });
      });

      var newPlacemark = new ymaps.Placemark(
        center,
        {
          hintContent: address,
          balloonContent: point.name,
        },
        {
          preset: "islands#blackStretchyIcon",
          draggable: true,
        }
      );

      newPlacemark.events.add("dragend", () => {
        let newCoord = newPlacemark.geometry.getCoordinates();
        ymaps.geocode(newCoord).then(function (res) {
          var firstGeoObject = res.geoObjects.get(0);
          address = firstGeoObject.getAddressLine();
          newPlacemark.properties.set({
            hintContent: firstGeoObject.getAddressLine(),
          });
        });

        this.initPolyline();
      });

      point.marker = newPlacemark;
      this.points.push(point);

      this.myMap.geoObjects.add(newPlacemark);

      this.initPolyline();
    },
    delPoint(point) {
      let mark = this.points.find((el) => el.id == point.id);
      this.myMap.geoObjects.remove(mark.marker);

      this.points = this.points.filter((el) => el.id !== point.id);
      this.initPolyline();
    },
    initPolyline() {
      if (this.points.length <= 1) {
        if (this.polyline !== null) this.myMap.geoObjects.remove(this.polyline);
        return;
      }

      let coords = [];

      for (let i = 0; i < this.points.length; i++) {
        const element = this.points[i];
        coords.push(element.marker.geometry._coordinates);
      }

      if (this.polyline !== null) this.myMap.geoObjects.remove(this.polyline);

      var myPolyline = new ymaps.Polyline(
        coords,
        {},
        {
          strokeColor: "#000000",
          strokeWidth: 2,
          strokeOpacity: 0.5,
        }
      );

      this.polyline = myPolyline;

      this.myMap.geoObjects.add(myPolyline);
    },
  },
  mounted() {
    ymaps.ready(this.init);
  },
};
</script>

<style>
body{
  background: #FAFCFF;
}
.container {
  margin: 0 20px;
  margin-top: 20px;
}
.page_points {
 display: flex;
 flex-wrap: wrap;
 justify-content: start;
 align-items: center;
 row-gap: 10px;
 margin-top: 20px;
}
.page__input {
  max-width: 250px;
  padding-top: 0;
  margin-right: 20px;
  margin-left: 20px;
}
.br{
  width: 100%;
}
.page_points h4 {
  margin-left: 20px;
}
.point {
  margin-bottom: 10px;
}
.point_list {
  margin: 0 20px;
  display: flex;
  gap: 10px;
}
</style>
