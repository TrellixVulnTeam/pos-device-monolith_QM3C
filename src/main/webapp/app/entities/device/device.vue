<template>
  <div>
    <h2 id="page-heading" data-cy="DeviceHeading">
      <span v-text="$t('posdevicerestapiApp.device.home.title')" id="device-heading">Devices</span>
      <div class="d-flex justify-content-end">
        <button class="btn btn-info mr-2" v-on:click="handleSyncList" :disabled="isFetching">
          <font-awesome-icon icon="sync" :spin="isFetching"></font-awesome-icon>
          <span v-text="$t('posdevicerestapiApp.device.home.refreshListLabel')">Refresh List</span>
        </button>
        <router-link :to="{ name: 'DeviceCreate' }" custom v-slot="{ navigate }">
          <button
            @click="navigate"
            id="jh-create-entity"
            data-cy="entityCreateButton"
            class="btn btn-primary jh-create-entity create-device"
          >
            <font-awesome-icon icon="plus"></font-awesome-icon>
            <span v-text="$t('posdevicerestapiApp.device.home.createLabel')"> Create a new Device </span>
          </button>
        </router-link>
      </div>
    </h2>
    <br />
    <div class="alert alert-warning" v-if="!isFetching && devices && devices.length === 0">
      <span v-text="$t('posdevicerestapiApp.device.home.notFound')">No devices found</span>
    </div>
    <div class="table-responsive" v-if="devices && devices.length > 0">
      <table class="table table-striped" aria-describedby="devices">
        <thead>
          <tr>
            <th scope="row" v-on:click="changeOrder('id')">
              <span v-text="$t('global.field.id')">ID</span>
              <jhi-sort-indicator :current-order="propOrder" :reverse="reverse" :field-name="'id'"></jhi-sort-indicator>
            </th>
            <th scope="row" v-on:click="changeOrder('name')">
              <span v-text="$t('posdevicerestapiApp.device.name')">Name</span>
              <jhi-sort-indicator :current-order="propOrder" :reverse="reverse" :field-name="'name'"></jhi-sort-indicator>
            </th>
            <th scope="row" v-on:click="changeOrder('description')">
              <span v-text="$t('posdevicerestapiApp.device.description')">Description</span>
              <jhi-sort-indicator :current-order="propOrder" :reverse="reverse" :field-name="'description'"></jhi-sort-indicator>
            </th>
            <th scope="row" v-on:click="changeOrder('price')">
              <span v-text="$t('posdevicerestapiApp.device.price')">Price</span>
              <jhi-sort-indicator :current-order="propOrder" :reverse="reverse" :field-name="'price'"></jhi-sort-indicator>
            </th>
            <th scope="row" v-on:click="changeOrder('deviceSize')">
              <span v-text="$t('posdevicerestapiApp.device.deviceSize')">Device Size</span>
              <jhi-sort-indicator :current-order="propOrder" :reverse="reverse" :field-name="'deviceSize'"></jhi-sort-indicator>
            </th>
            <th scope="row" v-on:click="changeOrder('image')">
              <span v-text="$t('posdevicerestapiApp.device.image')">Image</span>
              <jhi-sort-indicator :current-order="propOrder" :reverse="reverse" :field-name="'image'"></jhi-sort-indicator>
            </th>
            <th scope="row" v-on:click="changeOrder('deviceModel.name')">
              <span v-text="$t('posdevicerestapiApp.device.deviceModel')">Device Model</span>
              <jhi-sort-indicator :current-order="propOrder" :reverse="reverse" :field-name="'deviceModel.name'"></jhi-sort-indicator>
            </th>
            <th scope="row"></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="device in devices" :key="device.id" data-cy="entityTable">
            <td>
              <router-link :to="{ name: 'DeviceView', params: { deviceId: device.id } }">{{ device.id }}</router-link>
            </td>
            <td>{{ device.name }}</td>
            <td>{{ device.description }}</td>
            <td>{{ device.price }}</td>
            <td v-text="$t('posdevicerestapiApp.Size.' + device.deviceSize)">{{ device.deviceSize }}</td>
            <td>
              <a v-if="device.image" v-on:click="openFile(device.imageContentType, device.image)">
                <img
                  v-bind:src="'data:' + device.imageContentType + ';base64,' + device.image"
                  style="max-height: 30px"
                  alt="device image"
                />
              </a>
              <span v-if="device.image">{{ device.imageContentType }}, {{ byteSize(device.image) }}</span>
            </td>
            <td>
              <div v-if="device.deviceModel">
                <router-link :to="{ name: 'DeviceModelView', params: { deviceModelId: device.deviceModel.id } }">{{
                  device.deviceModel.name
                }}</router-link>
              </div>
            </td>
            <td class="text-right">
              <div class="btn-group">
                <router-link :to="{ name: 'DeviceView', params: { deviceId: device.id } }" custom v-slot="{ navigate }">
                  <button @click="navigate" class="btn btn-info btn-sm details" data-cy="entityDetailsButton">
                    <font-awesome-icon icon="eye"></font-awesome-icon>
                    <span class="d-none d-md-inline" v-text="$t('entity.action.view')">View</span>
                  </button>
                </router-link>
                <router-link :to="{ name: 'DeviceEdit', params: { deviceId: device.id } }" custom v-slot="{ navigate }">
                  <button @click="navigate" class="btn btn-primary btn-sm edit" data-cy="entityEditButton">
                    <font-awesome-icon icon="pencil-alt"></font-awesome-icon>
                    <span class="d-none d-md-inline" v-text="$t('entity.action.edit')">Edit</span>
                  </button>
                </router-link>
                <b-button
                  v-on:click="prepareRemove(device)"
                  variant="danger"
                  class="btn btn-sm"
                  data-cy="entityDeleteButton"
                  v-b-modal.removeEntity
                >
                  <font-awesome-icon icon="times"></font-awesome-icon>
                  <span class="d-none d-md-inline" v-text="$t('entity.action.delete')">Delete</span>
                </b-button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <b-modal ref="removeEntity" id="removeEntity">
      <span slot="modal-title"
        ><span id="posdevicerestapiApp.device.delete.question" data-cy="deviceDeleteDialogHeading" v-text="$t('entity.delete.title')"
          >Confirm delete operation</span
        ></span
      >
      <div class="modal-body">
        <p id="jhi-delete-device-heading" v-text="$t('posdevicerestapiApp.device.delete.question', { id: removeId })">
          Are you sure you want to delete this Device?
        </p>
      </div>
      <div slot="modal-footer">
        <button type="button" class="btn btn-secondary" v-text="$t('entity.action.cancel')" v-on:click="closeDialog()">Cancel</button>
        <button
          type="button"
          class="btn btn-primary"
          id="jhi-confirm-delete-device"
          data-cy="entityConfirmDeleteButton"
          v-text="$t('entity.action.delete')"
          v-on:click="removeDevice()"
        >
          Delete
        </button>
      </div>
    </b-modal>
    <div v-show="devices && devices.length > 0">
      <div class="row justify-content-center">
        <jhi-item-count :page="page" :total="queryCount" :itemsPerPage="itemsPerPage"></jhi-item-count>
      </div>
      <div class="row justify-content-center">
        <b-pagination size="md" :total-rows="totalItems" v-model="page" :per-page="itemsPerPage" :change="loadPage(page)"></b-pagination>
      </div>
    </div>
  </div>
</template>

<script lang="ts" src="./device.component.ts"></script>
