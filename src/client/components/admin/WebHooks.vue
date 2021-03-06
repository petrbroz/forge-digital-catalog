<template>
  <v-col cols="12">
    <v-card v-if="$store.state.isAdminUserLoggedIn">
      <v-card-title primary-title>
        <div>
          <h3 class="headline mb-0">
            WebHooks
          </h3>
        </div>
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="webhooks"
        item-key="hookId"
        class="elevation-1"
      >
        <template v-slot:item.action="{ item }">
          <v-icon
            small
            @click="deleteWebHook(item.id)"
          >
            {{ $t('general.delete') }}
          </v-icon>
        </template>
      </v-data-table>
      <v-card-actions>
        <v-btn
          color="primary"
          @click="setWebHook"
        >
          {{ $t('admin.newWebHook') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-col>
</template>

<script lang='ts'>
import { Component, Vue } from 'vue-property-decorator';
import config from '../../config';
import { validateSession } from '../../utils/utils';

@Component
export default class WebHooks extends Vue {

  protected alert: boolean = false;
  protected alertMessage: string = '';
  protected headers: any = [{
    align: 'left',
    sortable: false,
    text: 'Hook Id',
    value: 'id'
  }, {
    text: 'Tenant',
    value: 'tenant'
  }, {
    text: 'Event',
    value: 'event'
  }, {
    text: 'System',
    value: 'system'
  }, {
    text: 'Status',
    value: 'status'
  }, {
    text: 'Action',
    value: 'action'
  }];
  protected webhooks: string[] = [];

  beforeMount(): void {
    const loggedInSession = localStorage.getItem('loggedInSession');
    if (loggedInSession) {
      const retrievedSession = validateSession(loggedInSession);
      // detect if query param isAdminUserLoggedIn is true
      if (this.$route.query.isAdminUserLoggedIn || retrievedSession) {
        this.getWebHooks();
      }
    }
  }

  protected async deleteWebHook(hookId: string): Promise<void> {
    try {
      this.$store.dispatch('setDeleting', { webHook: true });
      const res = await this.$axios({
        method: 'DELETE',
        url: new URL(`/api/admin/webhooks/${hookId}`, config.koahost).href
      });
      if (res.status === 200) {
        this.$log.info(`... deleted WebHookId: ${hookId}`);
      }
    } catch (err) {
      this.alert = true;
      this.alertMessage = err;
    } finally {
      this.$store.dispatch('setDeleting', { webHook: false });
      this.getWebHooks();
    }
  }

  protected async setWebHook(): Promise<void> {
    try {
      this.$store.dispatch('setSaving', { newWebHook: true });
      const res = await this.$axios({
        method: 'POST',
        url: new URL(`/api/admin/webhooks`, config.koahost).href
      });
      if (res.status === 200) {
        this.$log.info('Saved new webhook');
      }
    } catch (err) {
      this.alert = true;
      this.alertMessage = err;
    } finally {
      this.$store.dispatch('setSaving', { newWebHook: false });
      this.getWebHooks();
    }
  }

  private async getWebHooks(): Promise<void> {
    try {
      this.$store.dispatch('setLoading', { webHooksInfo: true });
      const res = await this.$axios({
        method: 'GET',
        url: new URL(`/api/admin/webhooks`, config.koahost).href
      });
      if (res.status === 200) {
        const webHooksInfo = res.data.data;
        this.webhooks = webHooksInfo.map((val: any) => {
          return {
            event: val.event,
            id: val.hookId,
            status: val.status,
            system: val.system,
            tenant: val.tenant
          };
        });
      }
    } catch (err) {
      this.alert = true;
      this.alertMessage = err;
    } finally {
      this.$store.dispatch('setLoading', { webHooksInfo: false });
    }
  }

}
</script>

