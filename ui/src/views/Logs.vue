<!--
#
# Copyright (C) 2019 Nethesis S.r.l.
# http://www.nethesis.it - nethserver@nethesis.it
#
# This script is part of NethServer.
#
# NethServer is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License,
# or any later version.
#
# NethServer is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with NethServer.  If not, see COPYING.
#
-->

<template>
  <div>
    <h1>{{$t('logs.title')}}</h1>
    <form class="form-horizontal">
      <div class="form-group">
        <div class="col-xs-12 col-sm-3 col-md-2">
          <select id="selectLogPath" class="selectpicker form-control" v-model="view.path" v-on:change="handleLogs()">
            <option selected>/var/log/asterisk/full</option>
          </select>
        </div>
        <div class="col-xs-12 col-sm-6 col-md-8">
          <button
            type="button"
            v-on:click="toggleFollow()"
            class="btn btn-default"
          ><span v-bind:class="['fa', view.follow ? 'fa-stop':'fa-play']"></span>
          &nbsp;{{view.follow ? $t('logs.stop_follow_button') : $t('logs.start_follow_button')}}</button>
        </div>
      </div>
    </form>
    <form role="form" class="search-pf has-button form-horizontal" v-on:submit.prevent="">
      <div class="form-group has-clear">
        <div class="search-pf-input-group">
          <label for="search1" class="sr-only">Search</label>
          <input
              v-model.lazy="view.filter"
              v-on:change="handleLogs()"
              v-bind:placeholder="$t('logs.filter_label')"
              id="log-filter"
              class="filter form-control"
              type="search"
          >
          <button type="button" class="clear" aria-hidden="true"><span class="pficon pficon-close"></span></button>
        </div>
      </div>
      <div class="form-group">
        <button class="btn btn-primary" type="button"><span class="fa fa-search"></span></button>
      </div>
    </form>
    <div v-if="!view.logsLoaded" id="loader" class="spinner spinner-lg view-spinner"></div>
    <pre v-else id="logs-output" class="logs">{{view.logsContent}}</pre>
  </div>
</template>

<script>
export default {
  name: "Logs",
  mounted() {
    var context = this;
    window.jQuery('#selectLogPath').selectpicker();
    (function($) {
      $(document).ready(function() {
        // Hide the clear button if the search input is empty
        $(".search.has-clear .clear").each(function() {
          if (!$(this).prev('.form-control').val()) {
            $(this).hide();
          }
        });
        // Show the clear button upon entering text in the search input
        $(".search-pf .has-clear .form-control").keyup(function () {
          var t = $(this);
          t.next('button').toggle(Boolean(t.val()));
        });
        // Upon clicking the clear button, empty the entered text and hide the clear button
        $(".search-pf .has-clear .clear").click(function () {
          context.view.filter = "";
          context.handleLogs();
          $(this).prev('.form-control').focus();
          $(this).hide();
        });
      });
    })(window.jQuery);
    this.getLogs();
  },
  data() {
    return {
      view: {
        path: "/var/log/asterisk/full",
        logsLoaded: false,
        logsContent: "",
        follow: false,
        filter: "",
        lines: 100,
        process: null
      }
    };
  },
  beforeRouteLeave(to, from, next) {
    $(".modal").modal("hide");
    this.process.close()
    next();
  },
  methods: {
    toggleFollow() {
        this.view.follow = !this.view.follow
        this.handleLogs()
    },
    handleLogs() {
      this.view.logsContent = "";
      this.$forceUpdate();
      this.getLogs();
    },
    getLogs() {
      var context = this;
      this.process = nethserver.readLogs(
        {
          action: this.view.follow ? "follow" : "dump",
          lines: this.view.follow ? null : this.view.lines,
          mode: "file",
          filter: this.view.filter,
          paths: [this.view.path]
        },
        this.view.follow
          ? function(stream) {
              context.view.logsLoaded = true;

              context.view.logsContent += stream;

              document.getElementById(
                "logs-output"
              ).scrollTop = document.getElementById("logs-output").scrollHeight;
            }
          : null,
        function(success) {
          context.view.logsLoaded = true;
          context.view.logsContent = success;
          setTimeout(function() {
            if(document.getElementById("logs-output")) {
              document.getElementById(
                "logs-output"
              ).scrollTop = document.getElementById("logs-output").scrollHeight;
            }
          }, 100);

        },
        function(error) {
          context.view.logsLoaded = true;
          context.logsContent = error;
        },
        false
      );
    }
  }
};
</script>

<style scoped>
#logs-output {
    margin-top: 15px;
}

#logs-output:empty {
    display: none
}

.logs {
  max-height: 500px;
}
</style>
