<template>
  <v-container>
    <div>
      <v-row v-if="isRootElement">
        <v-col cols="12">
          <span class="text-subtitle-2 font-weight-medium grey--text text--darken-2">
            <v-icon left>mdi-book</v-icon>Deployment Information:
          </span>
        </v-col>
        <v-col cols="12" class="d-flex flex-column mx-8">
          <div>
            <span class="font-weight-medium"> Deployment Date: </span>
            {{ deploymentDate }}
          </div>
          <div>
            <span class="font-weight-medium"> Deployment Method: </span>
            {{ deploymentMethod }}
          </div>
          <div class="d-flex align-center">
            <span class="font-weight-medium mr-2"> Machines: </span>
            <SimpleMachineList chips :machines="machines" />
          </div>
        </v-col>
      </v-row>
      <v-row v-if="isRootElement">
        <v-col cols="12">
          <span class="text-subtitle-2 font-weight-medium grey--text text--darken-2">
            <v-icon left>mdi-book</v-icon>Extended Instance Information:
          </span>
        </v-col>
        <v-col cols="12" class="d-flex flex-column mx-8">
          <div>
            <span class="font-weight-medium"> Process ID: </span>
            {{ instanceInformation.processId }}
          </div>
          <div>
            <span class="font-weight-medium"> Process Instance ID: </span>
            {{ instanceInformation.processInstanceId }}
          </div>
          <div>
            <span class="font-weight-medium"> Process Version: </span>
            {{ instanceInformation.processVersion }}
          </div>
        </v-col>
      </v-row>
      <v-row v-else>
        <v-col cols="12">
          <span class="text-subtitle-2 font-weight-medium grey--text text--darken-2">
            <v-icon left>mdi-book</v-icon>General Activity Information:
          </span>
          <div class="d-flex flex-column mx-8">
            <div>
              <span class="font-weight-medium"> Activity ID: </span> {{ selectedElement.id }}
            </div>
            <div>
              <span class="font-weight-medium"> Activity Type: </span> {{ selectedElement.type }}
            </div>
          </div>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <v-expansion-panels ref="activity-advanced-panels" v-model="panels" flat multiple>
            <v-expansion-panel v-if="isRootElement" :disabled="adaptationItems.length === 0">
              <v-expansion-panel-header class="pa-0">
                <span class="text-subtitle-2 font-weight-medium grey--text text--darken-2">
                  <v-icon left>mdi-swap-horizontal</v-icon>Adaptation Log:
                </span>
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <div>
                  <v-treeview :items="adaptationItems" item-key="id">
                    <template v-slot:label="{ item }">
                      <div class="d-flex">
                        <span style="white-space: pre">{{ item.name }}: </span>
                        <span style="white-space: normal">
                          {{ item.value }}
                        </span>
                      </div>
                    </template>
                  </v-treeview>
                </div>
              </v-expansion-panel-content>
            </v-expansion-panel>
            <v-expansion-panel v-if="isRootElement">
              <v-expansion-panel-header class="pa-0">
                <span class="text-subtitle-2 font-weight-medium grey--text text--darken-2">
                  <v-icon left>mdi-database</v-icon>Variables:
                </span>
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <div>
                  <v-treeview :items="variableItems" item-key="id">
                    <template v-slot:label="{ item }">
                      <div class="d-flex">
                        <span style="white-space: pre">{{ item.name }}: </span>
                        <span style="white-space: normal">
                          {{ item.value }}
                        </span>
                      </div>
                    </template>
                  </v-treeview>
                </div>
              </v-expansion-panel-content>
            </v-expansion-panel>
            <v-expansion-panel
              id="activity-advanced-token-panel"
              :disabled="tokenItems.length === 0"
            >
              <v-expansion-panel-header class="pa-0">
                <span class="text-subtitle-2 font-weight-medium grey--text text--darken-2">
                  <v-icon left>mdi-book</v-icon>Token Information:
                </span>
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <div>
                  <v-treeview :items="tokenItems" item-key="id" :open.sync="openedTokens">
                    <template v-slot:label="{ item }">
                      <div class="d-flex">
                        <span style="white-space: pre">{{ item.name }}: </span>
                        <span style="white-space: normal">
                          {{ item.value }}
                        </span>
                      </div>
                    </template>
                  </v-treeview>
                </div>
              </v-expansion-panel-content>
            </v-expansion-panel>
            <v-expansion-panel :disabled="logItems.length === 0">
              <v-expansion-panel-header class="pa-0">
                <span class="text-subtitle-2 font-weight-medium grey--text text--darken-2">
                  <v-icon left>mdi-book</v-icon>Log Information:
                </span>
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <div>
                  <v-treeview :items="logItems" item-key="id">
                    <template v-slot:label="{ item }">
                      <div class="d-flex">
                        <span style="white-space: pre">{{ item.name }}: </span>
                        <span style="white-space: normal">
                          {{ item.value }}
                        </span>
                      </div>
                    </template>
                  </v-treeview>
                </div>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <slot name="additional-content"></slot>
        </v-col>
      </v-row>
    </div>
  </v-container>
</template>

<script>
import SimpleMachineList from '@/frontend/components/deployments/SimpleMachineList.vue';
export default {
  components: { SimpleMachineList },
  props: {
    deployment: Object,
    instance: Object,
    selectedVersion: Object,
    selectedElement: Object,
    selectedToken: Object,
  },
  data() {
    return {
      panels: [],
      openedTokens: [],
    };
  },
  computed: {
    isRootElement() {
      return this.selectedElement && this.selectedElement.type === 'bpmn:Process';
    },
    instanceInformation() {
      if (this.selectedElement && this.instance) {
        return this.instance;
      }
      return {};
    },
    version() {
      let version;
      if (this.instance) {
        version = this.deployment.versions.find(
          ({ version }) => version == this.instance.processVersion
        );
      } else if (this.selectedVersion) {
        version = this.selectedVersion;
      } else {
        version = this.deployment.versions[this.deployment.versions.length - 1];
      }
      return version;
    },
    deploymentDate() {
      if (this.version.deploymentDate) {
        return new Date(this.version.deploymentDate);
      }
      return '';
    },
    deploymentMethod() {
      return this.version.deploymentMethod || '';
    },
    machines() {
      if (this.instance) {
        return this.instance.machines;
      } else {
        return this.deployment.machines;
      }
    },
    adaptationItems() {
      if (this.selectedElement && this.instance) {
        return this.transformInfoIntoTreeViewStructure(this.instance.adaptationLog);
      }
      return [];
    },
    tokensOnSelectedElement() {
      if (this.selectedElement && this.instance) {
        const tokens = this.isRootElement
          ? this.instance.tokens
          : this.instance.tokens.filter(
              (token) => token.currentFlowElementId === this.selectedElement.id
            );
        return tokens;
      }
      return [];
    },
    tokenItems() {
      if (this.selectedElement && this.instance) {
        const tokensTreeViewStructure = this.tokensOnSelectedElement.map((token) => {
          return {
            id: `Token ID (${token.tokenId})`,
            name: 'Token ID',
            value: token.tokenId,
            // get every entry of token expect for its tokenID to prevent redundancy
            children: this.transformInfoIntoTreeViewStructure(token, token.tokenId).filter(
              ({ name }) => name !== 'tokenId'
            ),
          };
        });
        return tokensTreeViewStructure;
      }
      return [];
    },
    logItems() {
      if (this.selectedElement && this.instance) {
        const selectedElementLogs = this.isRootElement
          ? this.instance.log
          : this.instance.log.filter((log) => log.flowElementId === this.selectedElement.id);
        return this.transformInfoIntoTreeViewStructure(selectedElementLogs);
      }
      return [];
    },
    variableItems() {
      if (this.selectedElement && this.instance) {
        const variablesTreeViewStructure = [];
        Object.entries(this.instance.variables).forEach(([key, value]) => {
          variablesTreeViewStructure.push({
            id: key,
            name: key,
            value: value.value,
            children: this.transformInfoIntoTreeViewStructure(value),
          });
        });
        return variablesTreeViewStructure;
      }
      return [];
    },
  },
  methods: {
    transformInfoIntoTreeViewStructure(info, itemIdAddition) {
      const treeViewToken = [];

      Object.entries(info).forEach(([key, value]) => {
        const namePrefix = Array.isArray(info) ? 'Entry ' : '';
        if (value && typeof value === 'object' && Object.keys(value).length > 0) {
          treeViewToken.push({
            id: itemIdAddition ? `${key} (${itemIdAddition})` : key,
            name: `${namePrefix}${key}`,
            children: this.transformInfoIntoTreeViewStructure(value),
          });
        } else {
          treeViewToken.push({
            id: itemIdAddition ? `${key} (${itemIdAddition})` : key,
            name: `${namePrefix}${key}`,
            value: value,
          });
        }
      });

      return treeViewToken;
    },
    async showTokenInfo(tokenIds) {
      await this.$nextTick();
      // find the index of the token panel in the element containing all panels
      const tokenPanelIndex = this.$refs['activity-advanced-panels'].$children.findIndex(
        (child) => child.$attrs.id === 'activity-advanced-token-panel'
      );
      // make sure that the token panel is the one thats opened
      this.panels = [tokenPanelIndex];
      // open the token(s)
      if (Array.isArray(tokenIds)) {
        this.openedTokens = tokenIds.map((tokenId) => `Token ID (${tokenId})`);
      } else {
        this.openedTokens = [`Token ID (${tokenIds})`];
      }
    },
  },
  watch: {
    selectedElement: {
      handler(newSelection) {
        this.panels = [];
        this.openedTokens = [];

        const tokenIds = this.tokensOnSelectedElement.map((token) => token.tokenId);
        if (!this.isRootElement && tokenIds.length > 0) {
          this.showTokenInfo(tokenIds);
        }
      },
      immediate: true,
    },
    selectedToken: {
      async handler(newSelection) {
        if (newSelection) {
          await this.showTokenInfo(newSelection.tokenId);
        }
      },
      immediate: true,
    },
  },
};
</script>

<style scoped>
.v-treeview-node__root {
  min-height: 24px !important;
}

.v-expansion-panel-header {
  min-height: 40px !important;
}

#activity-advanced-token-panel
  >>> .v-treeview-node__toggle:not(div.v-treeview-node__children .v-treeview-node__toggle) {
  display: none !important;
}
</style>
