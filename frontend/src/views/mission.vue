<template>
  <Editor v-if="editorMode" @goBack="switchEditorStatus"></Editor>
  <v-layout
    v-else
    class="main-container"
    column
    fill-height
    justify-space-between
  >
    <v-navigation-drawer
      app
      :mini-variant="this.$vuetify.breakpoint.smAndDown && miniVariant"
      permanent
      touchless
    >
      <v-layout column dense fill-height justify-space-between>
        <div id="topSidebar">
          <v-list v-if="this.$vuetify.breakpoint.smAndDown" dense>
            <v-list-item @click="miniVariant = !miniVariant">
              <v-list-item-icon>
                <v-icon color="blue">mdi-format-list-bulleted</v-icon>
              </v-list-item-icon>
              <v-list-item-title></v-list-item-title>
            </v-list-item>
          </v-list>

          <NavigationCommands :accessStatus="accessStatus" />
          <v-divider></v-divider>
          <DroneCommands
            v-if="isDroneCommandsEnabled()"
            :accessStatus="accessStatus"
            :droneid="getSelectedDrone()"
          />
          <v-list nav>
            <v-list-item @click="isLogsMenuOpen = !isLogsMenuOpen">
              <v-list-item-icon>
                <v-icon color="blue">mdi-note-text</v-icon>
              </v-list-item-icon>
              <v-list-item-title v-if="!isLogsMenuOpen"
                >Ouvrir logs</v-list-item-title
              >
              <v-list-item-title v-if="isLogsMenuOpen"
                >Fermer logs</v-list-item-title
              >
            </v-list-item>
            <v-list-item
              :disabled="!isUserControlling()"
              @click="switchEditorStatus"
            >
              <v-list-item-icon>
                <v-icon color="blue">mdi-laptop</v-icon>
              </v-list-item-icon>
              <v-list-item-title>Programmer</v-list-item-title>
            </v-list-item>
          </v-list>
        </div>
        <div id="bottomSidebar" justify-end>
          <v-divider></v-divider>
          <MissionCommands
            v-if="isUserControlling()"
            :accessStatus="accessStatus"
            :maps="maps"
          />

          <v-list dense nav>
            <v-list-item v-if="isConnected()">
              <v-list-item-icon>
                <v-icon color="blue">mdi-access-point-network</v-icon>
              </v-list-item-icon>
              <v-list-item-title>Connecté</v-list-item-title>
            </v-list-item>
            <v-list-item v-else>
              <v-list-item-icon>
                <v-icon color="red">mdi-access-point-network-off</v-icon>
              </v-list-item-icon>
              <v-list-item-title>Déconnecté</v-list-item-title>
            </v-list-item>
          </v-list>
        </div>
      </v-layout>
    </v-navigation-drawer>

    <div id="topContent">
      <v-item-group>
        <v-container fluid style="border-bottom: 1px solid grey">
          <v-row dense no-gutters style="flex-wrap: nowrap; overflow-x: auto">
            <v-col class="d-flex" cols="auto">
              <v-item v-slot="{toggle}">
                <v-card
                  :class="chosenOption === -1 ? 'selected' : 'unselected'"
                  :elevation="0"
                  :style="'display: flex; '"
                  @click="
                    toggle();
                    setSelected(-1);
                  "
                >
                  <v-card-text
                    id="general"
                    :class="
                      chosenOption === -1
                        ? 'selected centered-card'
                        : 'unselected centered-card'
                    "
                    >General</v-card-text
                  >
                </v-card>
              </v-item>
            </v-col>
            <v-col
              v-for="(droneData, index) in droneList"
              :key="index"
              cols="auto"
            >
              <v-item v-slot="{toggle}">
                <drone-data-card
                  :droneData="droneData"
                  :selected="chosenOption === index ? true : false"
                  :userControlling="isUserControlling()"
                  @click="
                    toggle();
                    setSelected(index);
                  "
                  @deleteDrone="deleteDrone(index)"
                ></drone-data-card>
              </v-item>
            </v-col>
            <v-col class="d-flex" cols="auto">
              <v-item v-if="isUserControlling() && !getMissionStatus()">
                <v-card
                  id="add-drone"
                  class="centered-card unselected"
                  elevation="0"
                  @click.stop="isDroneMenuOpen = true"
                >
                  <v-card-text>+</v-card-text>
                </v-card>
              </v-item>
            </v-col>
          </v-row>
        </v-container>
      </v-item-group>

      <div>
        <Map :droneList="droneList" :indexDrone="indexDrone" :maps="maps" />
      </div>
    </div>

    <DroneMenu
      :currentDroneList="droneList"
      :droneList="droneList"
      :isDroneMenuOpen="isDroneMenuOpen"
      :isMissionSimulated="accessStatus.isMissionSimulated"
      @addDrone="addDrone"
      @setDroneMenuOpen="setDroneMenuOpen"
    />

    <div v-if="isLogsMenuOpen" justify-end style="width: 100%">
      <v-btn @click="isLogsMenuOpen = !isLogsMenuOpen">
        <v-icon color="black"> mdi-close </v-icon>Logs
      </v-btn>
      <LogsInterface />
    </div>
  </v-layout>
</template>

<style scoped>
.v-btn--active.no-active:not(:hover)::before {
  opacity: 0 !important;
}

.card-container {
  min-width: 205px;
}

.main-container {
  min-width: min-content;
}

.logs-menu {
  position: absolute;
  bottom: 0;
  max-width: 100vw;
  width: 100%;
}

.centered-card {
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 0 !important;
}

#general {
  border-top-left-radius: 4px !important;
  border-bottom-left-radius: 4px !important;
  font-size: 16px;
  border: 1px solid #1976d211;
}

#add-drone {
  border-top-right-radius: 4px !important;
  border-bottom-right-radius: 4px !important;
}

#LogTitle {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

#logs {
  border: 10px;
  float: left;
  width: 100%;
  overflow-x: auto;
}
</style>

<script lang="ts">
import {Component, Vue} from 'vue-property-decorator';
import DroneCommands from '@/components/drone_commands.vue';
import MissionCommands from '@/components/mission_commands.vue';
import NavigationCommands from '@/components/navigation_commands.vue';
import Map from '@/components/map.vue';
import DroneDataCard from '@/components/drone_data_card.vue';
import DroneMenu from '@/components/drone_menu.vue';
import {
  SOCKETIO_LIMITED_ACCESS,
  SOCKETIO_DRONE_STATUS,
} from '@/communication/server_constants';
import {AccessStatus} from '@/communication/access_status';
import {
  DroneData,
  NewDroneData,
  DEFAULT_DRONE_DATA,
} from '@/communication/drone';
import RemoteCommandOutput from '@/components/remote_command_output.vue';
import LogsInterface from '@/components/logs_interface.vue';
import {ServerCommunication} from '@/communication/server_communication';
import Editor from '@/components/editor.vue';
import {ACCESSOR} from '@/store/index';

@Component({
  components: {
    DroneCommands,
    MissionCommands,
    NavigationCommands,
    DroneMenu,
    Map,
    LogsInterface,
    DroneDataCard,
    RemoteCommandOutput,
    Editor,
  },
})
export default class Mission extends Vue {
  public editorMode = false;
  public miniVariant = true;
  public attemptedLimitedConnexion = false;
  public dialog = false;
  public isDroneMenuOpen = false;
  public isLogsMenuOpen = false;
  public droneList: DroneData[] = [];
  public maps: HTMLCanvasElement[] = [];
  public isMissionEnding = false;
  public accessStatus = {
    isMissionSimulated: false,
    isUserControlling: false,
  } as AccessStatus;

  private indexDrone = 0;
  private chosenOption = -1;

  constructor() {
    super();

    window.addEventListener('keydown', (event: KeyboardEvent) => {
      if (event.ctrlKey && event.key === '=') {
        event.preventDefault();
        this.setDroneMenuOpen(true);
      }
    });
  }

  public getMissionStatus(): boolean {
    return ACCESSOR.missionStatus.isMissionStarted;
  }

  public deleteDrone(index: number): void {
    if (this.chosenOption >= index) {
      this.chosenOption = -1;
    }
    this.droneList.splice(index, 1);

    ServerCommunication.setDrone(
      this.getNewDrones(),
      this.accessStatus.isMissionSimulated
    );
  }

  public setSelected(index: number): void {
    this.chosenOption = index;
    this.indexDrone = index + 1;
  }

  public setDroneMenuOpen(value: boolean): void {
    this.isDroneMenuOpen = value;
  }

  public getNewDrones(): NewDroneData[] {
    return this.droneList.map(droneData => {
      return {
        name: droneData.name,
        startingXPos: droneData.startingXPos,
        startingYPos: droneData.startingYPos,
        startingOrientation: droneData.startingOrientation,
      } as NewDroneData;
    });
  }

  public addDrone(drone: NewDroneData): void {
    this.droneList.push({...DEFAULT_DRONE_DATA, ...drone} as DroneData);
    this.droneList[this.droneList.length - 1].xPos = drone.startingXPos;
    this.droneList[this.droneList.length - 1].yPos = drone.startingYPos;
    this.setStartingPoint(drone, this.droneList.length - 1);
    ServerCommunication.setDrone(
      this.getNewDrones(),
      this.accessStatus.isMissionSimulated
    );
  }

  public isDroneCommandsEnabled(): boolean {
    return this.chosenOption !== -1 && this.accessStatus.isUserControlling;
  }

  public isUserControlling(): boolean {
    return this.accessStatus.isUserControlling;
  }

  public getSelectedDrone(): string {
    if (this.chosenOption !== -1 && this.droneList.length > 0) {
      return this.droneList[this.chosenOption].name;
    }
    return '';
  }

  public isConnected(): boolean {
    return SOCKETIO_LIMITED_ACCESS.connected;
  }

  public switchEditorStatus(): void {
    this.editorMode = !this.editorMode;
  }

  private setStartingPoint(newDroneData: NewDroneData, index: number) {
    this.droneList[index].startingXPos = newDroneData.startingXPos;
    this.droneList[index].startingYPos = newDroneData.startingYPos;
    this.droneList[index].startingOrientation =
      newDroneData.startingOrientation;
  }

  private beforeCreate(): void {
    SOCKETIO_LIMITED_ACCESS.on(
      'update_status',
      (accessStatus: AccessStatus, callback) => {
        this.accessStatus = accessStatus;
        if (callback !== undefined) callback();
      }
    );

    SOCKETIO_LIMITED_ACCESS.on('disconnect', () => {
      this.accessStatus.isUserControlling = false;
    });

    SOCKETIO_DRONE_STATUS.on(
      'update_drone_status',
      (droneDataList: Array<DroneData>) => {
        droneDataList.forEach(droneData => {
          for (let i = 0; i < this.droneList.length; i++) {
            if (this.droneList[i].name === droneData.name) {
              Vue.set(this.droneList, i, {
                ...this.droneList[i],
                ...droneData,
              } as DroneData);
            }
          }
        });
      }
    );

    SOCKETIO_LIMITED_ACCESS.on('droneList', (droneList: Array<DroneData>) => {
      this.droneList = droneList.map(droneValue => {
        return {...DEFAULT_DRONE_DATA, ...droneValue} as DroneData;
      });
    });

    SOCKETIO_LIMITED_ACCESS.open();
    SOCKETIO_DRONE_STATUS.open();
  }

  private destroyed() {
    SOCKETIO_LIMITED_ACCESS.removeAllListeners().close();
    SOCKETIO_DRONE_STATUS.removeAllListeners().close();
  }
}
</script>
