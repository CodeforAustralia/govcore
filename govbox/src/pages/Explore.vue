<template>
    <div>

        <section>
            <h1 class="title">My Workspace</h1>

            <b-tabs type="is-toggle" expanded v-model="activeTab">
                <b-tab-item label="Documents" icon="google-photos" @click="selectTab('documents')"></b-tab-item>
                <b-tab-item label="Graph" icon="library-music" @click="selectTab('graph')"></b-tab-item>

            </b-tabs>
        </section>

        <div v-if="activeTab === 0">
            <div class="columns is-mobile">

                <div class="column is-2">

                    <div class="field is-grouped">
                        <div class="control">
                        <a class="button is-info" @click="addDocument()">
                            <span class="icon is-small">
                                <i class="fas fa-plus"></i>
                            </span>
                            <span>Document</span>
                        </a>
                        </div>
                        <div class="control">
                        <a class="button is-success" @click="refreshDocuments()">
                            <span class="icon is-small">
                                <i class="fas fa-sync"></i>
                            </span>
                            
                        </a>
                        </div>
                    </div>
                    
                </div>
                
                <div class="column is-10">                     
                    <h1 class="title">{{stemplate}}</h1>
                    
                </div>


            </div>
            <div class="columns is-mobile">
                <div class="column is-2">

                    <aside class="menu">
                        <ul class="menu-list">
                            <h1 class="subtitle">Templates</h1>

                            <li>
                                <a class="button is-success" @click="addTemplate()">
                                    <span class="icon is-small">
                                        <i class="fas fa-plus"></i>
                                    </span>
                                    <span>Add</span>
                                </a>
                            </li>
                            <li>
                                <a class="button is-primary" @click="editTemplate()">
                                    <span class="icon is-small">
                                        <i class="fas fa-edit"></i>
                                    </span>
                                    <span>Edit</span>
                                </a>
                            </li>
                            <li v-for="tem in templates" :value="tem.id" :key="tem.id">

                                <a @click="getContent(tem.type,tem.id)">{{ tem.type}} </a>
                            
                            </li>
                            
                           

                        </ul>


                    </aside>

                </div>
                <div class="column is-10">
                    <div class="tile is-ancestor">
                        <div class="tile is-parent">
                            <article class="tile is-child">
                                <div class="content">

                                    <div class="content">

                                        <b-table :data="documents" detailed detail-key="id">

                                            <template slot-scope="props">

                                                <b-table-column field="id" label="ID" sortable>
                                                    {{ props.row.id }}
                                                </b-table-column>

                                                <b-table-column field="type" label="Type" sortable>
                                                    {{ props.row.type }}
                                                </b-table-column>

                                                <b-table-column field="bucket" label="Bucket" sortable>
                                                    {{ props.row.bucket }}
                                                </b-table-column>

                                            </template>

                                            <template slot="detail" slot-scope="props">

                                                <article class="media">

                                                    <div class="media-content">
                                                        <div class="content">
                                                            <div class="field is-grouped">
                                                                <div class="control">
                                                                    <a class="button is-primary" @click="updateDocument(props.row)">
                                                                        <span class="icon is-small ">
                                                                            <i class="fas fa-edit"></i>
                                                                        </span>
                                                                        <span>Edit</span>
                                                                    </a>
                                                                </div>
                                                                <div class="control">
                                                                    <a class="button is-danger" @click="deleteDocument(props.row.id)">
                                                                        <span class="icon is-small">
                                                                            <i class="fas fa-minus"></i>
                                                                        </span>
                                                                        <span>Delete</span>
                                                                    </a>
                                                                </div>
                                                            </div>

                                                            <pretty-print :value="props.row"></pretty-print>

                                                        </div>
                                                    </div>
                                                </article>
                                            </template>
                                        </b-table>


                                    </div>
                                </div>
                            </article>
                        </div>
                    </div>

                 
                </div>
            </div>


        </div>
        <div v-if="activeTab === 1"></div>

        <b-loading :is-full-page="true" :active.sync="isLoading" :can-cancel="true"></b-loading>
        <b-modal :active.sync="isCardModalActive" :width="640" scroll="keep">
            <div class="card">
               
                <div class="card-content">
                        <vue-form-json-schema :model="model" :schema="schema" :ui-schema="uiSchema" v-on:change="onChange"
                            v-on:state-change="onChangeState" v-on:validated="onValidated">
                        </vue-form-json-schema>

                        <h4>Document</h4>
                        <pretty-print :value="model"></pretty-print>
                      

                        <div class="field is-grouped">
                            <div class="control">
                                <a class="button is-success" @click="save(model)">
                                    <span class="icon is-small ">
                                        <i class="fas fa-edit"></i>
                                    </span>
                                    <label>Save</label>
                                </a>
                            </div>
                            
                        </div>
                </div>
            </div>
        </b-modal>
    </div>
</template>

<script>
    // Import VueFormJsonSchema
    import VueFormJsonSchema from 'vue-form-json-schema';
    import PrettyPrint from '../components/pretty-print';
    import govcoreapi from '../services/govcoreapi';

    const apicore = new govcoreapi();


    export default {
        name: 'Explore-Component',
        components: {
            'vue-form-json-schema': VueFormJsonSchema,
            'pretty-print': PrettyPrint
        },
        data() {
            return {
                templates: [],
                documents: [],
                edges: [],
                stemplate: "",
                idtemplate: "",
                isLoading: false,
                valid: false,
                isCardModalActive:false,
                creating:false,
                activeTab: 0,                
                model: {},
                state: {},               
                schema: {},
                uiSchema: [],
            }
        },
        created() {
            this.getTemplates();
                    
        },
        methods: {

            onChange(value) {
              
                this.model = value;
            },
            onChangeState(value) {
                this.state = value;
            },
            onValidated(value) {
                this.valid = value;
            },
            async addDocument() {
                this.isCardModalActive=true;
                this.model={};
                this.creating=true;

            },
            async addTemplate() {
                this.isCardModalActive=true;

            },
            async editTemplate() {
                this.isCardModalActive=true;
                this.creating=false;

            },
            async refreshDocuments(){
                this.getDocuments(this.stemplate, this.idtemplate);
                this.getTemplates();
            },
            async save(docum){

            this.isLoading = true;

                    if(this.creating){

                        await apicore.createDocument(docum).then(response => {

                                        
                                        // eslint-disable-next-line 
                                        var something = response;
                                        this.$toast.open({
                                            message: 'Document created succesfully!',
                                            type: 'is-success'
                                        });
                                        this.refreshDocuments();
                                        this.isLoading = false;
                                    })// eslint-disable-next-line 
                                        .catch(e => {
                                            this.isLoading = false;
                                            this.$toast.open({
                                                duration: 4000,
                                                message: "Something's not good." + e,
                                                position: 'is-top',
                                                type: 'is-danger'
                                            });
                                        });

                    }
                    else
                    {

                    await apicore.updateDocument(docum).then(response => {

                                        
                                        // eslint-disable-next-line 
                                        var something = response;
                                        this.$toast.open({
                                            message: 'Document updated succesfully!',
                                            type: 'is-success'
                                        });
                                        this.refreshDocuments();
                                        this.isLoading = false;

                                    })// eslint-disable-next-line 
                                        .catch(e => {
                                            this.isLoading = false;
                                            this.$toast.open({
                                                duration: 4000,
                                                message: "Something's not good." + e,
                                                position: 'is-top',
                                                type: 'is-danger'
                                            });
                                        });
                    }
            },
            async updateDocument(docum) {
          
                this.isCardModalActive=true;
                this.model=docum;

            },
            async deleteDocument(docum) {


                    this.$dialog.confirm({
                    title: 'Deleting',
                    message: 'Are you sure you want to <b>delete</b> this document? ',
                    confirmText: 'Delete',
                    type: 'is-danger',
                    hasIcon: true,
                    onConfirm: () => {
                        this.isLoading = true;
                         apicore.deleteTemplate(docum).then(response => {
                            this.getDocuments(this.stemplate, this.idtemplate);
                            // eslint-disable-next-line 
                            var something = response;
                            this.isLoading = false;
                        })// eslint-disable-next-line 
                        .catch(e => {
                        this.isLoading = false;
                            this.$toast.open({
                                duration: 4000,
                                message: "Something's not good." + e,
                                position: 'is-top',
                                type: 'is-danger'
                            });
                        });                         
                    }
                })
            },

            async getContent(stemplate, idtemplate){
                

                if(stemplate==="graph_edge") 
                    this.getGraphEdges(stemplate,idtemplate);
                else
                    this.getDocuments(stemplate,idtemplate)


            },
            async getDocuments(stemplate, idtemplate) {

                this.stemplate = stemplate;
                this.idtemplate = idtemplate;
                this.isLoading = true;

                await apicore.getDocuments(this.stemplate,"entities").then(response => {
                    this.documents = response.data


                    //LOADING SCHEMA
                    apicore.getDocument(this.idtemplate).then(response => {                       
                     
                        this.schema = response.data.schema;           

                        var fields =Object.keys(response.data.schema.properties);                        
                        this.uiSchema=[];
                        fields.forEach( field => {                    
                            this.uiSchema.push({component:"input",model:field,  fieldOptions: { on: ['input']  }})
                        });    

                        this.isLoading = false;

                    })// eslint-disable-next-line 
                        .catch(e => {
                            this.isLoading = false;
                            this.$toast.open({
                                duration: 3000,
                                message: "Something's not good, try again",
                                position: 'is-top',
                                type: 'is-danger'
                            });
                        });




                })// eslint-disable-next-line 
                    .catch(e => {
                        this.isLoading = false;
                        this.$toast.open({
                            duration: 3000,
                            message: "Something's not good, try again",
                            position: 'is-top',
                            type: 'is-danger'
                        });
                    });
                    
                  
            },
            async getGraphEdges(stemplate,idtemplate){

                this.isLoading = true;
                this.stemplate = stemplate;
                this.idtemplate = idtemplate;

                await apicore.getGraphEdges().then(response => {
                    
                    this.isLoading = false;
                    this.documents = response.data


                    //LOADING SCHEMA
                    apicore.getDocument(this.idtemplate).then(response => {                       
                     
                        this.schema = response.data.schema;           

                        var fields =Object.keys(response.data.schema.properties);                        
                        this.uiSchema=[];
                        fields.forEach( field => {                    
                            this.uiSchema.push({component:"input",model:field,  fieldOptions: { on: ['input']  }})
                        });    

                        this.isLoading = false;

                    })// eslint-disable-next-line 
                        .catch(e => {
                            this.isLoading = false;
                            this.$toast.open({
                                duration: 3000,
                                message: "Something's not good, try again",
                                position: 'is-top',
                                type: 'is-danger'
                            });
                        });

                    
                    

                })// eslint-disable-next-line 
                    .catch(e => {
                        this.isLoading = false;
                        this.$toast.open({
                            duration: 3000,
                            message: "Something's not good, try again",
                            position: 'is-top',
                            type: 'is-danger'
                        });
                    });
            }
            
            ,
            async getTemplates() {
                this.isLoading = true;
                await apicore.getTemplates().then(response => {
                    this.templates = response.data;
                    this.isLoading = false;
                    this.stemplate = this.templates[0].type;

                    this.idtemplate = this.templates[0].id;
                    this.getDocuments(this.stemplate, this.idtemplate);
                })// eslint-disable-next-line 
                    .catch(e => {
                        this.isLoading = false;
                        this.$toast.open({
                            duration: 3000,
                            message: "Something's not good, try again",
                            position: 'is-top',
                            type: 'is-danger'
                        });
                    });

            }
        }

    }

</script>