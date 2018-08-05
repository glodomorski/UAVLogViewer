<template>
  <div id='vuewrapper'>
    <nav class="navbar navbar-dark fixed-top bg-dark flex-md-nowrap p-0 shadow">
      <a class="navbar-brand col-sm-3 col-md-2 mr-0" href="#">Mav Viewer</a>

    </nav>

  <div class="container-fluid">
    <div class="row">
      <nav class="col-md-2 d-none d-md-block bg-light sidebar">
        <div class="sidebar-sticky">
          <ul class="nav flex-column">
            <li v-for="item in message_types" class="nav-item" :key="'li' + item">
              <a :key="'a' + item" class="nav-link" href="#" @click="plot(item)">
                <span :key="item" data-feather="file"></span>
                {{item}}
              </a>
            </li>
          </ul>

          <h6 class="sidebar-heading d-flex justify-content-between align-items-center px-3 mt-4 mb-1 text-muted">
          </h6>

          <Dropzone ref="dropzone" v-on:messages="updateData"/>

        </div>
      </nav>

      <main role="main" class="col-md-9 ml-sm-auto col-lg-10 px-4">
        <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3">
          <Plotly v-on:attitude="updateAttitude" v-bind:plot-data="current_data"/>
        </div>
        <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3">
          <Cesium ref="cesium" v-bind:trajectory="current_trajectory"/>
        </div>
      </main>
    </div>
  </div>
  </div>
</template>

<script>
import Dropzone from './Dropzone'
import Plotly from './Plotly'
import Cesium from './Cesium'

export default {
  name: 'Home',
  data () {
    return {
      messages: {},
      message_types: [],
      current_data: [],
      current_trajectory: [],
      time_trajectory: {},
      time_attitude: {}
    }
  },
  methods: {
    updateData () {
      this.messages = this.$refs.dropzone.messages
      this.message_types = Object.keys(this.messages).sort()
      this.current_trajectory = this.extractTrajectory(this.messages)
      this.time_attitude = this.extractAttitudes(this.messages)
    },
    plot (item) {
      this.current_data = this.messages[item]
    },
    extractTrajectory (messages) {
      let gpsData = messages['GLOBAL_POSITION_INT']
      let trajectory = []
      for (var pos of gpsData) {
        if (pos.lat !== 0) {
          trajectory.push(pos.lon)
          trajectory.push(pos.lat)
          trajectory.push(pos.relative_alt)
          this.time_trajectory[pos.time_boot_ms] = [pos.lon, pos.lat, pos.relative_alt]
        }
      }
      return trajectory
    },
    extractAttitudes (messages) {
      let attitudeMsgs = messages['ATTITUDE']
      let attitudes = {}
      for (var att of attitudeMsgs) {
        attitudes[att.time_boot_ms] = [att.roll, att.pitch, att.yaw]
      }
      return attitudes
    },
    updateAttitude (time) {
      this.$refs.cesium.showAttitude(time, this.time_trajectory, this.time_attitude)
    }
  },
  components: {
    Dropzone,
    Plotly,
    Cesium}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  body {
    font-size: .875rem;
  }

  .feather {
    width: 16px;
    height: 16px;
    vertical-align: text-bottom;
  }

  /*
   * Sidebar
   */

  .sidebar {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    z-index: 100; /* Behind the navbar */
    padding: 48px 0 0; /* Height of navbar */
    box-shadow: inset -1px 0 0 rgba(0, 0, 0, .1);
  }

  .sidebar-sticky {
    position: relative;
    top: 0;
    height: calc(100vh - 48px);
    padding-top: .5rem;
    overflow-x: hidden;
    overflow-y: auto; /* Scrollable contents if viewport is shorter than content. */
  }

  @supports ((position: -webkit-sticky) or (position: sticky)) {
    .sidebar-sticky {
      position: -webkit-sticky;
      position: sticky;
    }
  }

  .sidebar .nav-link {
    font-weight: 500;
    color: #333;
  }

  .sidebar .nav-link .feather {
    margin-right: 4px;
    color: #999;
  }

  .sidebar .nav-link.active {
    color: #007bff;
  }

  .sidebar .nav-link:hover .feather,
  .sidebar .nav-link.active .feather {
    color: inherit;
  }

  .sidebar-heading {
    font-size: .75rem;
    text-transform: uppercase;
  }

  /*
   * Content
   */

  [role="main"] {
    padding-top: 48px; /* Space for fixed navbar */
  }

  /*
   * Navbar
   */

  .navbar-brand {
    padding-top: .75rem;
    padding-bottom: .75rem;
    font-size: 1rem;
    background-color: rgba(0, 0, 0, .25);
    box-shadow: inset -1px 0 0 rgba(0, 0, 0, .25);
  }

  .navbar .form-control {
    padding: .75rem 1rem;
    border-width: 0;
    border-radius: 0;
  }

  .form-control-dark {
    color: #fff;
    background-color: rgba(255, 255, 255, .1);
    border-color: rgba(255, 255, 255, .1);
  }

  .form-control-dark:focus {
    border-color: transparent;
    box-shadow: 0 0 0 3px rgba(255, 255, 255, .25);
  }

</style>