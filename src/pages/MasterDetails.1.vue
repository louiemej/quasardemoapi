<template>
    <div class="q-pa-md">
        <q-table
            title="Invoices"
            :data="data"
            :columns="columns"
            row-key="id"
            :filter="filter"
            :loading="loading"
        >
            <!-- q-table top part -->
            <template v-slot:top>
                <span class="text-h5" style="margin-right: 12px;">Invoices</span>
                <q-btn
                    color="positive"
                    @click="onCreate()"
                    icon="add" size="sm"
                    class="btn-action"
                />
                <q-space/>
                <q-input borderless dense debounce="300" color="primary" v-model="filter">
                    <template v-slot:append>
                        <q-icon name="search"/>
                    </template>
                </q-input>
            </template>
            
            <!-- q-table body part -->
            <template v-slot:body="props">
                <q-tr :props="props">
                    <q-td key="id" :props="props">{{ props.row.id }}</q-td>
                    <q-td key="soldTo" :props="props">{{ props.row.sold_to }}</q-td>
                    <q-td key="businessStyle" class="text-ellipsis" :props="props">
                        {{ props.row.business_style }}
                    </q-td>
                    <q-td key="address" class="text-ellipsis" :props="props">
                        {{ props.row.address }}
                        <q-popup-edit v-model="props.row.address">
                            <span>{{ props.row.address }}</span>
                        </q-popup-edit>
                    </q-td>
                    <q-td key="action" :props="props" class="q-gutter-xs" style="min-width: 140px;">
                        <q-btn color="primary" class="btn-action" icon="visibility" size="sm" @click="onView(props.row)"/>
                        <q-btn color="secondary" class="btn-action" icon="edit" size="sm" @click="onUpdate(props.row)"/>
                        <q-btn color="deep-orange" class="btn-action" icon="delete" size="sm" @click="onDelete(props.row)"/>
                    </q-td>
                </q-tr>
            </template>
        </q-table>

        <!-- create dialog -->
        <q-dialog v-model="create_dialog" transition-show="flip-down" transition-hide="flip-up">
            <q-card style="width: 900px; max-width: 80vw;">
                <q-card-section class="bg-indigo-6">
                <div class="text-h6">View Invoice Details</div>
                </q-card-section>
                <q-card-section style="padding-top: 16px;">
                    <div class="text-center text-h4">
                        TRUMPH MOTORCYCLE CORPORATION<br />
                        <div class="text-caption" style="line-height: 100%;">
                            TORIL BRANCH: Saavedra Street, Brgy, Toril, Davao City Davao Del Sur<br />
                            VAT Reg. TIN: 284-708-476-024<br />
                            MAIN BRANCH: TMC Bldg., Allah Valley Drive, Libertad Surallah, South Cotabato
                        </div>
                        <br />
                        <div class="row q-pm-a-lg" style="margin-left: 20px; margin-right: 20px;">
                            <table style="width: 100%">
                                <tbody>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Sold To: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 89%; border-bottom: 1px solid black; padding: 0px; height: 20px !important;" colspan="3">
                                            <q-input
                                                v-model="invoiceDetails.sold_to"
                                                type="text"
                                                :error=errors.sold_to
                                                :error-message=errors.sold_to_message
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                            />
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Bus. Style: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 49%; border-bottom: 1px solid black">
                                            <q-input
                                                v-model="invoiceDetails.business_style"
                                                type="text"
                                                :error=errors.business_style
                                                :error-message=errors.business_style_message
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                            />
                                        </td>
                                        <td class="text-body1 text-center" style="width: 11%;">Date: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 29%; border-bottom: 1px solid black">

                                            <q-input
                                                v-model="invoiceDetails.created_at"
                                                mask="date"
                                                :rules="['date']"
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                                >
                                                <template v-slot:append>
                                                    <q-icon name="event" class="cursor-pointer">
                                                        <q-popup-proxy ref="qDateProxy" transition-show="scale" transition-hide="scale">
                                                            <q-date landscape v-model="invoiceDetails.created_at" @input="() => $refs.qDateProxy.hide()" />
                                                        </q-popup-proxy>
                                                    </q-icon>
                                                </template>
                                            </q-input>

                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Address: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 39%; border-bottom: 1px solid black" colspan="3">
                                            <q-input
                                                v-model="invoiceDetails.address"
                                                type="text"
                                                :error=errors.address
                                                :error-message=errors.address_message
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                            />
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <br />
                    <q-markup-table separator="cell">
                        <thead>
                            <tr style="width: 100%;">
                                <th style="width: 35%; text-align: center;">Description</th>
                                <th style="width: 20%; text-align: center;">Quantity</th>
                                <th style="width: 20%; text-align: center;">Price</th>
                                <th style="width: 20%; text-align: center;">Total</th>
                                <th style="width: 5%; text-align: center;">Action</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(line, index) in lines" v-bind:key="index">
                                <td>
                                    <q-input
                                    v-model="line.description"
                                    :rules="[val => !!val]"
                                    :error=description[index]
                                    @focus="description[index] = false"
                                    />
                                </td>
                                <td>
                                    <q-input
                                    v-model="line.quantity"
                                    @keyup="computeTotal"
                                    @focus="quantity[index] = false"
                                    :rules="[val => !!val]"
                                    :error=quantity[index]
                                    />
                                </td>
                                <td>
                                    <q-input
                                    v-model="line.price"
                                    @keyup="computeTotal"
                                    @focus="price[index] = false"
                                    :rules="[val => !!val]"
                                    :error=price[index]
                                    />
                                </td>
                                <td>
                                    <span class="text-subtitle1 text-blue-10">
                                        {{ line.quantity * line.price }}
                                    </span>
                                </td>
                                <td class="text-center">
                                    <q-btn @click="removeLine(index)" size="xs" icon="delete" round />
                                </td>
                            </tr>
                        </tbody>
                    </q-markup-table>
                    <div class="row">
                        <div class="col-3 offset-9 q-pa-md q-ma-md text-center" style="border: 1px solid #ccc">
                        {{ grand_total }}
                        </div>
                    </div>
                    <br />
                    <q-btn color="positive" icon="add" size="sm" @click="addLine" label="Add new record"/>                    
                </q-card-section>
                <q-card-actions align="right">
                <q-btn flat label="OK" color="teal" @click="saveArray()"/>
                <q-btn flat label="Close" color="primary" @click="closeArray()" />
                </q-card-actions>
            </q-card>
        </q-dialog>

        <!-- view dialog -->
        <q-dialog v-model="view_dialog" transition-show="flip-down" transition-hide="flip-up">
            <q-card style="width: 900px; max-width: 80vw;">
                <q-card-section class="bg-indigo-6">
                <div class="text-h6">View Invoice Details</div>
                </q-card-section>
                <q-card-section style="padding-top: 16px;">
                    <div class="text-center text-h4">
                        TRUMPH MOTORCYCLE CORPORATION<br />
                        <div class="text-caption" style="line-height: 100%;">
                            TORIL BRANCH: Saavedra Street, Brgy, Toril, Davao City Davao Del Sur<br />
                            VAT Reg. TIN: 284-708-476-024<br />
                            MAIN BRANCH: TMC Bldg., Allah Valley Drive, Libertad Surallah, South Cotabato
                        </div>
                        <br />
                        <div class="row q-pm-a-lg" style="margin-left: 20px; margin-right: 20px;">
                            <table style="width: 100%">
                                <tbody>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Sold To: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 89%; border-bottom: 1px solid black; padding: 0px; height: 20px !important;" colspan="3">
                                            {{ invoiceDetails.sold_to }}
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Bus. Style: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 49%; border-bottom: 1px solid black">
                                            {{ invoiceDetails.business_style }}
                                        </td>
                                        <td class="text-body1 text-center" style="width: 11%;">Date: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 29%; border-bottom: 1px solid black">
                                            {{ invoiceDetails.created_at }}
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Address: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 39%; border-bottom: 1px solid black" colspan="3">
                                            {{ invoiceDetails.address }}
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <br />
                    <q-markup-table separator="cell">
                        <thead>
                            <tr style="width: 100%;">
                                <th style="width: 35%; text-align: center;">Description</th>
                                <th style="width: 20%; text-align: center;">Quantity</th>
                                <th style="width: 20%; text-align: center;">Price</th>
                                <th style="width: 20%; text-align: center;">Total</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(line, index) in lines" v-bind:key="index">
                                <td>
                                    {{ line.description }}
                                </td>
                                <td>
                                    {{ line.quantity }}
                                </td>
                                <td>
                                    {{ line.price }}
                                </td>
                                <td>
                                    <span class="text-subtitle1 text-blue-10">
                                        {{ line.quantity * line.price }}
                                    </span>
                                </td>
                            </tr>
                        </tbody>
                    </q-markup-table>
                    <div class="row">
                        <div class="col-3 offset-9 q-pa-md q-ma-md text-center" style="border: 1px solid #ccc">
                        {{ grand_total }}
                        </div>
                    </div>              
                </q-card-section>
                <q-card-actions align="right">
                <q-btn flat label="Close" color="primary" @click="closeView()" />
                </q-card-actions>
            </q-card>
        </q-dialog>

        <!-- prompt info dialog -->
        <q-dialog v-model="alertSuccess">
            <q-card class="bg-teal text-white">
                <q-card-section>
                <div class="text-h6">Success</div>
                </q-card-section>

                <q-card-section>
                Process completed successfully.
                </q-card-section>

                <q-card-actions align="right">
                <q-btn flat label="OK" color="white" v-close-popup />
                </q-card-actions>
            </q-card>
        </q-dialog>

        <q-dialog v-model="alertFailure">
            <q-card class="bg-deep-orange-14 text-white">
                <q-card-section>
                <div class="text-h6">Failed</div>
                </q-card-section>

                <q-card-section>
                Process interupted.
                </q-card-section>

                <q-card-actions align="right">
                <q-btn flat label="OK" color="white" v-close-popup />
                </q-card-actions>
            </q-card>
        </q-dialog>

        <!-- question dialog -->

        <q-dialog v-model="delete_dialog" transition-show="flip-down" transition-hide="flip-up">
            <q-card style="width: 300px;">
                <q-card-section class="bg-indigo-6">
                <div class="text-h6 text-white">Confirm Action</div>
                </q-card-section>

                <q-card-section>
                    <br />
                Confirm Delete? <br />
                <span class="text-deep-orange">{{ this.invoiceDetails.sold_to }}</span> <span class="text-blue" style="margin-left: 10px;">[ {{ this.invoiceDetails.id }} ]</span>
                </q-card-section>

                <q-card-actions align="right">
                <q-btn flat label="Delete" color="negative" @click="execDelete()" />
                <q-btn flat label="Cancel" color="primary" v-close-popup />
                </q-card-actions>
            </q-card>
        </q-dialog>


        <!-- update dialog -->
        <q-dialog v-model="update_dialog" transition-show="flip-down" transition-hide="flip-up">
            <q-card style="width: 900px; max-width: 80vw;">
                <q-card-section class="bg-yellow-6">
                <div class="text-h6">Update Master Details</div>
                </q-card-section>
                <q-card-section style="padding-top: 16px;">
                    <div class="text-center text-h4">
                        TRUMPH MOTORCYCLE CORPORATION<br />
                        <div class="text-caption" style="line-height: 100%;">
                            TORIL BRANCH: Saavedra Street, Brgy, Toril, Davao City Davao Del Sur<br />
                            VAT Reg. TIN: 284-708-476-024<br />
                            MAIN BRANCH: TMC Bldg., Allah Valley Drive, Libertad Surallah, South Cotabato
                        </div>
                        <br />
                        <div class="row q-pm-a-lg" style="margin-left: 20px; margin-right: 20px;">
                            <table style="width: 100%">
                                <tbody>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Sold To: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 89%; border-bottom: 1px solid black; padding: 0px; height: 20px !important;" colspan="3">
                                            <q-input
                                                v-model="invoiceDetails.sold_to"
                                                type="text"
                                                :error=errors.sold_to
                                                :error-message=errors.sold_to_message
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                            />
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Bus. Style: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 49%; border-bottom: 1px solid black">
                                            <q-input
                                                v-model="invoiceDetails.business_style"
                                                type="text"
                                                :error=errors.business_style
                                                :error-message=errors.business_style_message
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                            />
                                        </td>
                                        <td class="text-body1 text-center" style="width: 11%;">Date: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 29%; border-bottom: 1px solid black">

                                            <q-input
                                                v-model="invoiceDetails.created_at"
                                                mask="date"
                                                :rules="['date']"
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                                >
                                                <template v-slot:append>
                                                    <q-icon name="event" class="cursor-pointer">
                                                        <q-popup-proxy ref="qDateProxy" transition-show="scale" transition-hide="scale">
                                                            <q-date landscape v-model="invoiceDetails.created_at" @input="() => $refs.qDateProxy.hide()" />
                                                        </q-popup-proxy>
                                                    </q-icon>
                                                </template>
                                            </q-input>

                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="text-body1 text-left" style="width: 11%;">Address: </td>
                                        <td class="invoice-input text-body1 text-left" style="width: 39%; border-bottom: 1px solid black" colspan="3">
                                            <q-input
                                                v-model="invoiceDetails.address"
                                                type="text"
                                                :error=errors.address
                                                :error-message=errors.address_message
                                                :input-style="{
                                                    fontSize: '14pt',
                                                    padding: '0'
                                                }"
                                            />
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <br />
                    <q-markup-table separator="cell">
                        <thead>
                            <tr style="width: 100%;">
                                <th style="width: 35%; text-align: center;">Description</th>
                                <th style="width: 20%; text-align: center;">Quantity</th>
                                <th style="width: 20%; text-align: center;">Price</th>
                                <th style="width: 20%; text-align: center;">Total</th>
                                <th style="width: 5%; text-align: center;">Action</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(line, index) in lines" v-bind:key="index">
                                <td>
                                    <q-input
                                    v-model="line.description"
                                    :error=description[index]
                                    />
                                    <q-tooltip v-if="description[index]" content-class="bg-red" anchor="top left" self="center left" >
                                        Here I am!
                                    </q-tooltip>
                                </td>
                                <td>
                                    <q-input
                                    v-model="line.quantity"
                                    @keyup="computeTotal"
                                    :error=quantity[index]
                                    />
                                </td>
                                <td>
                                    <q-input
                                    v-model="line.price"
                                    @keyup="computeTotal"
                                    :error=price[index]
                                    />
                                </td>
                                <td>
                                    <span class="text-subtitle1 text-blue-10">
                                        {{ line.quantity * line.price }}
                                    </span>
                                </td>
                                <td class="text-center">
                                    <q-btn @click="removeLine(index)" size="xs" icon="delete" round />
                                </td>
                            </tr>
                        </tbody>
                    </q-markup-table>
                    <div class="row">
                        <div class="col-3 offset-9 q-pa-md q-ma-md text-center" style="border: 1px solid #ccc">
                        {{ grand_total }}
                        </div>
                    </div>
                    <br />
                    <q-btn color="positive" icon="add" size="sm" @click="addLine" label="Add new record"/>                    
                </q-card-section>
                <q-card-actions align="right">
                <q-btn flat label="OK" color="teal" @click="updateArray()"/>
                <q-btn flat label="Close" color="primary" @click="closeArray()" />
                </q-card-actions>
            </q-card>
        </q-dialog>

    </div>
</template>

<script>
import axios from "axios";
export default {
    data() {
        return {
            loading: false,
            filter: "",
            data: [],
            invoiceDetails: {
                id: '',
                sold_to: '',
                business_style: '',
                address: '',
                created_at: '',
                updated_at: ''
            },
            columns: [
                {
                    name: "id",
                    label: "Id",
                    align: "left",
                    field: "id  ",
                    sortable: true
                },
                {
                    name: "soldTo",
                    label: "Sold To",
                    align: "left",
                    field: "sold_to",
                    sortable: true
                },
                {
                    name: "businessStyle",
                    label: "Business Style",
                    align: "left",
                    field: "business_style",
                    sortable: true
                },
                {
                    name: "address",
                    label: "Address",
                    align: "left",
                    field: "address",
                    sortable: true
                },
                { name: "action", align: "center", label: "Action" }
            ],
            create_dialog: false,
            view_dialog: false,
            delete_dialog: false,
            update_dialog: false,
            alertSuccess: false,
            alertFailure: false,
            errors: {
            },
            lines: [],
            grand_total: '',
            description: {},
            quantity: {},
            price: {},
            payment: {}
        }
    },
    methods: {
        onCreate() {
            this.create_dialog = true;
            this.addLine();
        },
        // execCreate() {
        //     console.log(this.invoiceDetails.sold_to);
        //     console.log(this.invoiceDetails.business_style);
        //     console.log(this.invoiceDetails.address);
        //     console.log(this.invoiceDetails.created_at);
        //     console.log(this.invoiceDetails.updated_at);
        // },
        onView(row) {
            this.view_dialog = true;
            console.log(row);
            this.invoiceDetails.sold_to = row.sold_to;
            this.invoiceDetails.business_style = row.business_style;
            this.invoiceDetails.address = row.address;
            this.invoiceDetails.created_at = row.created_at;
            this.lines = row.items;

            console.log(this.invoiceDetails.created_at);


            this.grand_total = 0;
            var x;
            for(x in row.items) {
                this.grand_total += row.items[x]['quantity'] * row.items[x]['price'];
            }


        },
        onUpdate(row) {
            this.update_dialog = true;
            this.invoiceDetails.id = row.id;
            this.invoiceDetails.sold_to = row.sold_to;
            this.invoiceDetails.business_style = row.business_style;
            this.invoiceDetails.address = row.address

            this.payment.id = row.payment.id;
            this.payment.invoice_id = row.payment.invoice_id;
            this.payment.receipt_no = row.payment.receipt_no;
            this.payment.in_payment_of = row.payment.in_payment_of;
            this.payment.amount = row.payment.amount;
            this.payment.status_id = row.payment.status_id;

            var today = new Date(row.created_at);
            var current_year = today.getFullYear();
            var current_month = (today.getMonth() + 1) < 10 ? ('0' + (today.getMonth() + 1)) : (today.getMonth() + 1);
            var current_day = (today.getDate()) < 10 ? ('0' + (today.getDate())) : (today.getDate());
            var date = current_year + '/' + current_month + '/' + current_day;

            this.invoiceDetails.created_at = date;

            var x;
            for (x in row.items) {
                this.addLine();
                this.lines[x]['id'] = row.items[x]['id'];
                this.lines[x]['description'] = row.items[x]['description'];
                this.lines[x]['quantity'] = row.items[x]['quantity'];
                this.lines[x]['price'] = row.items[x]['price'];
            }

            console.log(this.lines);

        },
        onDelete(row) {
            this.delete_dialog = true;
            this.invoiceDetails.id = row.id;
            this.invoiceDetails.sold_to = row.sold_to;
        },
        execDelete() {
            axios.delete("http://demoapi.loc/api/invoice/" + this.invoiceDetails.id,
                { data: JSON.stringify({ id: this.invoiceDetails.id }) }
            )
            .then(response => {
                this.delete_dialog = false;
                this.alertSuccess = true;
                this.clearInputs();
            })
            .then(response=> {
                axios.get("http://demoapi.loc/api/invoices")
                .then(response => {
                    this.data = response["data"];
                })
                .catch(error => console.log(error));
            })
            .catch(error => {
                console.log(error);
                this.alertFailure = true;
            });
        },
        formatDate(date) {
            var d = new Date(date),
                month = '' + (d.getMonth() + 1),
                day = '' + d.getDate(),
                year = d.getFullYear();

            if (month.length < 2) month = '0' + month;
            if (day.length < 2) day = '0' + day;

            return [year, month, day].join('-');
        },
        addLine() {
            let checkEmptyLines = this.lines.filter(line => line.number === null);
            if (checkEmptyLines.length >= 1 && this.lines.length > 0) return
                this.lines.push({
                    description: null,
                    quantity: null,
                    price: null,
                });
            // console.log(this.lines);
        },
        removeLine (lineId) {
            if (!this.blockRemoval) {
                this.lines.splice(lineId, 1);
                this.computeTotal();
            }
        },
        computeTotal() {
            this.grand_total = 0;
            var x;
            for(x in this.lines) {
                this.grand_total += this.lines[x]['quantity'] * this.lines[x]['price'];
            }
        },
        closeView() {
            this.clearErrors();
            this.clearInputs();
            this.view_dialog = false;
        },
        closeArray() {
            this.clearErrors();
            this.clearInputs();
            this.create_dialog = false;
            this.update_dialog = false;
        },
        clearErrors() {
            this.errors = [];
            this.description = [];
            this.quantity = [];
            this.price = [];
        },
        clearInputs() {
            var today = new Date();
            var current_year = today.getFullYear();
            var current_month = (today.getMonth() + 1) < 10 ? ('0' + (today.getMonth() + 1)) : (today.getMonth() + 1);
            var current_day = (today.getDate()) < 10 ? ('0' + (today.getDate())) : (today.getDate());
            var date = current_year + '/' + current_month + '/' + current_day;
            this.invoiceDetails.created_at = date;

            this.invoiceDetails.id = '';
            this.invoiceDetails.sold_to = '';
            this.invoiceDetails.business_style = '';
            this.invoiceDetails.address = '';
            this.errors = {},
            this.lines = [],
            this.grand_total = '',
            this.description = {},
            this.quantity = {},
            this.price = {}
        },
        saveArray() {
            this.clearErrors();
            let checkEmptyLines = this.lines.filter(line => line.number === null);

            axios.post("http://demoapi.loc/api/invoice", JSON.stringify({
                invoice: this.invoiceDetails, lines: this.lines, 
            }))
            .then(response => {
                this.create_dialog = false;
                this.clearInputs();
            })
            .then(response=> {
                axios.get("http://demoapi.loc/api/invoices")
                .then(response => {
                    this.data = response["data"];
                })
                .catch(error => console.log(error));
            })
            .then(response=> {
                this.loading = false;
                this.alertSuccess = true;
                this.clearInputs();
            })
            .catch(error => { 
                var error_logs = error.response.data.errors;
                var errors = [];
                for (var key in error_logs) {
                    var subkey = key.split('.');
                    if(subkey[0] == 'invoice') {
                        errors[subkey[1]] = error_logs.hasOwnProperty(key);
                        // errors[subkey[1] + '_message'] = error.response.data.errors[key][0];
                    }
                    if(subkey[0] == 'lines') {
                        if(subkey[2] == 'description') {
                            this.description[subkey[1]] = error_logs.hasOwnProperty(key);
                        }
                        if(subkey[2] == 'quantity') {
                            this.quantity[subkey[1]] = error_logs.hasOwnProperty(key);
                        }
                        if(subkey[2] == 'price') {
                            this.price[subkey[1]] = error_logs.hasOwnProperty(key);
                        }
                    }
                }
                this.errors = errors;
            }); 
        },
        updateArray() {
            this.clearErrors();
            let checkEmptyLines = this.lines.filter(line => line.number === null);

            console.log(this.lines);
            console.log(this.invoiceDetails);
            console.log(this.payment);

            axios.put("http://demoapi.loc/api/invoice/" + this.invoiceDetails.id, JSON.stringify({
                invoice: this.invoiceDetails,
                lines: this.lines,
                payment: this.payment
            })) 
            .then(response => {
                this.update_dialog = false;
                this.clearInputs();
            })
            .then(response=> {
                axios.get("http://demoapi.loc/api/invoices")
                .then(response => {
                    this.data = response["data"];
                })
                .catch(error => console.log(error));
            })
            .then(response=> {
                this.loading = false;
                this.alertSuccess = true;
                this.clearInputs();
            })
            .catch(error => { 
                console.log(error)
                var error_logs = error.response.data.errors;
                var errors = [];
                for (var key in error_logs) {
                    var subkey = key.split('.');
                    if(subkey[0] == 'invoice') {
                        errors[subkey[1]] = error_logs.hasOwnProperty(key);
                        // errors[subkey[1] + '_message'] = error.response.data.errors[key][0];
                    }
                    if(subkey[0] == 'lines') {
                        if(subkey[2] == 'description') {
                            this.description[subkey[1]] = error_logs.hasOwnProperty(key);
                        }
                        if(subkey[2] == 'quantity') {
                            this.quantity[subkey[1]] = error_logs.hasOwnProperty(key);
                        }
                        if(subkey[2] == 'price') {
                            this.price[subkey[1]] = error_logs.hasOwnProperty(key);
                        }
                    }
                }
                this.errors = errors;
            }); 
        },

    },
    created() {
        axios.get("http://demoapi.loc/api/invoices")
        .then(response => {
            this.data = response["data"];
            console.log(this.data)
        })
        .catch(error => console.log(error));
        var today = new Date();
        var current_year = today.getFullYear();
        var current_month = (today.getMonth() + 1) < 10 ? ('0' + (today.getMonth() + 1)) : (today.getMonth() + 1);
        var current_day = (today.getDate()) < 10 ? ('0' + (today.getDate())) : (today.getDate());
        var date = current_year + '/' + current_month + '/' + current_day;
        this.invoiceDetails.created_at = date;
    }
}
</script>

<style scoped>

table {
    width: 100%;
}
td {
    max-width: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

td:nth-child(1) {
    width: 15%;
}

td:nth-child(2) {
    width: 10%;
}

td:nth-child(3) {
    width: 10%;
}

/* td:nth-child(4) {
    max-width: 20%;
} */

td:nth-child(5) {
    width: 25%;
}

td > div {
    height: 25px;
    padding: 0px;
}
</style>
