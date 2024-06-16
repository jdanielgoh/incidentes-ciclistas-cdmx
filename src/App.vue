<script setup>
import Header from "./components/navegacion/Header.vue";

import Footer from "./components/navegacion/Footer.vue";
import Mapa from "./components/Mapa.vue";
</script>

<template>
      <Header></Header>

  <div class="contenedor">
    <div class="contenedor ancho-lectura">
      <h1>ACCIDENTES CICLISTAS EN LA CIUDAD DE MÉXICO </h1>
      <p>
        En este mapa es resultado de una colaboración con Itxchel Lamadrid para
        el documental
        <a href="https://www.youtube.com/watch?v=Uq2i7jVnbwo" target="_blank"
          >Extracción inmediata</a
        >. En él se visualiza la infraestructura ciclista en la Ciudad de
        México, así como los hechos de tránsito que han involucrado a ciclistas
        a partir de 2018.
      </p>
      <p>
        Selecciona una opción de visualización, ya sea "Mapa de puntos" para ver
        la ubicación exacta de cada accidente y así poder distinguir y ajustar
        la distancia a la infraestructura ciclista de los accidentes, o bien, la
        opción "Mapa de calor" para ver las zonas con mayor densidad de
        accidentes.
      </p>
    </div>
    <div class="contenedor ancho-viz"><Mapa /></div>
    <div class="contenedor ancho-lectura">
      <p><b>Fuentes:</b></p>
      <ul>
        <li>
          <a
            href="https://datos.cdmx.gob.mx/dataset/hechos-de-transito-reportados-por-ssc-base-ampliada-no-comparativa"
          >
            Hechos de tránsito registrados por la SSC (serie de datos ampliada,
            no comparativa) </a
          >. Versión con última actualización del 6 de julio de 2023
        </li>
        <li>
          <a
            href="https://datos.cdmx.gob.mx/dataset/infraestructura-vial-ciclista"
          >
            Infraestructura vial ciclista </a
          >. Versión con última actualización del 7 de diciembre de 2023
        </li>
      </ul>

      <p>
        <b>*Nota importante:</b> La base de datos ofical que se utilizó tiene
        errores, se intentó contactar a las dependencias responsables por varios
        medios. Finalmente tuve respuesta el 12 de diciembre del 2023, en donde
        se me notificó que se revisaría el error. Al 19 de diciembre de 2023 no
        hay actualización, por cual los datos referentes a los ciclistas occisos
        pueden ser incorrectos.
      </p>
      <h2>Extracción inmediata</h2>
      <iframe
        width="560"
        height="315"
        style="width: 100%"
        src="https://www.youtube.com/embed/Uq2i7jVnbwo?si=irgpBvI5QJog-Cnh"
        title="YouTube video player"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
        allowfullscreen
      ></iframe>
    </div>
  </div>
  <Footer></Footer>

</template>

<style scoped lang="scss">
#app {
  width: 100%;
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
</style>
