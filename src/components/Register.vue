<template>
<div id="signUp">
		
	<div id="parent" class="panel" v-if="step == 1">
		<div class="titleBar">
			<router-link :to="{ name: 'login'}" class="back yipp-yipp_back"></router-link> {{ label.header_reg }}
		</div>
		
		
		<form action="" method="post" class="middle-area semi" v-on:submit.prevent="register">
		
			<div class="error" v-if="error_message">{{ error_message }}</div>
		
			<div class="selection-group">
				<input type="radio" id="father"  name="parents" value="father" v-model="parent_gender">
				<label for="father" class="rLeft">{{ label.reg_father }}</label>
				<input type="radio" id="mother"  name="parents" value="mother" v-model="parent_gender">
				<label for="mother" class="rRight">{{ label.reg_mother }}</label>
			</div>

			<div class="inputFields">
				
				<label>{{ label.reg_name }}</label>
				<input type="text" v-bind:placeholder="label.reg_name_placeholder" v-model="parent_name">

				<label>{{ label.reg_email }}</label>
				<input type="email" v-bind:placeholder="label.reg_email_placeholder" v-model="parent_email">

				<label>{{ label.reg_password }}</label>
				<input type="password" v-bind:placeholder="label.password_placeholder" v-model="parent_password">
			
			</div>

		
			<div class="bottom-area">
				<button class="button-red-medium">{{ label.reg_btn }}<span v-if="loading" class="loading"></span></button>
			</div>
			
		</form>

	</div>

	<div id="child" class="panel" v-else-if="step == 2">

		<div class="titleBar"><a href="javascript:void(0);" v-on:click="back" class="back yipp-yipp_back"></a> {{ label.reg_header_child }}</div>
		
		<form action="" method="post" class="middle-area semi" v-on:submit.prevent="addChild">
		
			<div class="error" v-if="error_message">{{ error_message }}</div>
		
			<div class="selection-group">
				<input type="radio" id="son"  name="child" value="son" v-model="child_gender">
				<label for="son" class="rLeft">{{ label.reg_son }}</label>
				<input type="radio" id="daughter"  name="child" value="daughter" v-model="child_gender">
				<label for="daughter" class="rRight">{{ label.reg_daugther }}</label>
			</div>

			<div class="inputFields">
				
				<label>{{ label.reg_child_name }}</label>
				<input type="text" v-bind:placeholder="label.reg_child_name_placeholder" v-model="child_name">

			</div>

			<div class="inputFields">

				<label>{{ label.reg_child_birthday }}</label>
				<ul id="date-group">
					<li><input type="tel" placeholder="DD" class="date-group" v-model="child_bday_d"></li>
					<li><input type="tel" placeholder="MM" class="date-group" v-model="child_bday_m"></li> 
					<li><input type="tel" placeholder="YYYY" class="date-group" v-model="child_bday_y"></li>
				</ul>
			
			</div>
			
			<div class="bottom-area">
				<button class="button-red-medium">{{ label.reg_btn }}<span v-if="loading" class="loading"></span></button>
			</div>
			
		</form>


	</div>
	
	<div id="photo" class="panel" v-else-if="step == 3">

		<div class="titleBar"><a href="javascript:void(0);" v-on:click="back" class="back yipp-yipp_back"></a> {{ label.reg_header_photo }}</div>

		<form action="" method="post" enctype="multipart/form-data" v-on:submit.prevent="addPicture">
			<div class="error" v-if="error_message">{{ error_message }}</div>
			
			<div class="inputFields">
				
				<input type="hidden" role="uploadcare-uploader" name="uploadedfile" id="uploadPic" />
				<!-- <input type="file" name="uploadedfile"  accept="image/*" id="uploadPic" capture> -->
				<label for="uploadPic" class="uploadPic">+</label>

			</div>

			<h3>{{ label.reg_photo_choose_msg }} {{ child_name }}!</h3>

			<div class="bottom-area">
				<input type="submit" value="label.reg_btn" class="button-red-medium"><span v-if="loading" class="loading"></span>
				<a href="javascript:void(0);" v-on:click="showLastStep" >{{ label.reg_photo_btn_nopic }}</a>
			</div>
			
		</form>

	</div>
						
	<div id="photoAdded" class="panel" v-else-if="step == 4">

		<div class="titleBar"><a href="javascript:void(0);" v-on:click="back" class="back yipp-yipp_back"></a> Add photo2</div>


		<div class="inputFields">
			
			<div class="picHolder"><img class="avatar" v-bind:src="child_image"></div>
				
			<h3>{{ label.reg_photo_rechoose_msg }}</h3>
			<a href="javascript:void(0);" v-on:click="showLastStep">{{ label.reg_photo_rechoose_upload }}</a>
	                         
		</div>                 
                                                                                                                                                                                                                                                                                                                                                                                                                 
		<div class="bottom-area">
			<a href="javascript:void(0);" v-on:click="showLastStep" class="button-red-medium">next</a>
		</div>
	</div>
	
	<div id="starten" class="panel" v-else-if="step == 5">
				
			<div class="placer middle">
						
				<p>{{ label.reg_finish }} <strong>Yipp <br> app!</strong></p>		
						
			</div> 							
			
			<div class="bottom-area">
				<router-link :to="{ name: 'timeline'}" class="button-white-medium">{{ label.reg_finish_btn }}</router-link>
			</div>
							
	</div>
								
</div>                                                                                                                                        
</template>


<script>
import locale from '../api/locale'
import {router} from '../index'
import config from '../config'
import auth from '../api/auth'
import register from '../api/register'
import $ from 'jquery'
import uploadcare from 'uploadcare-widget';

export default {
    data() {
        return {
            step: 1,
            error: false,
			error_message: '',
			parent_name: '',
			parent_gender: '',
			parent_email: '',
			parent_password: '',
			child_name: '',
			child_gender: '',
			child_bday_d: '',
			child_bday_m: '',
			child_bday_y: '',
			child_image: '',
			loading: false,
			label: {}
        }
    },
    created: function() {
    	
    },

    mounted: function () {
    	this.loadLabels();
    	auth.check();

    	if (auth.user) {
			if (auth.user.hasChild()) {
				this.showAddChildPhoto();
			} else {
				this.showAddChild();
			}
		}
    },

    methods: {
    	loadLabels: function () {
            var that = this;
            locale.label(this, config.api.lang, function (response) {
                that.label = response;
            }, function (msg, response) {
                that.logError(msg);
            });
        },
        showParentForm: function () {
    		this.resetError();
			this.step = 1;
    	},
        showAddChild: function () {
        	if (!auth.user) {
        		this.showParentForm();
        		return;
        	}

        	if (auth.user.hasChild()) {
        		this.showAddChildPhoto();
        		return;
        	}

        	this.resetError();
			this.step = 2;
    	},
    	showAddChildPhoto: function () {
    		if (!auth.user.hasChild()) {
        		this.showAddChild();
        		return;
        	}

        	this.resetError();
			this.step = 3;

			var that = this;

			setTimeout(function(){
                var input = $('[role=uploadcare-uploader]');
                var widget = uploadcare.Widget(input);
                
                widget.onChange(function(file) {
					if (file) {
						file.done(function(info) {
							this.loading = true;
							var credentials = {
					          	image: info.originalUrl,
					          	image_id: info.uuid,
					          	user_id: auth.user.get('id')
					        }

					        that.child_image = info.cdnUrl;

					        register.addPhoto(that, credentials, function(msg, response) {
					        	auth.user.get('child').setImage(info.cdnUrl)
					        	auth.saveUser(auth.user);
					        	that.showAddChildPhotoAgain();
					        }, function(msg, response) {
					        	that.logError(msg);
					        })
						});
					};
				});

            }, 1);
    	},
    	showAddChildPhotoAgain: function() {
    		this.resetError();
			this.step = 4;
    	},
        showLastStep: function () {
        	if (!auth.user) {
        		this.showParentForm();
        		return;
        	}

        	if (!auth.user.hasChild()) {
        		this.showAddChild();
        		return;
        	}

			this.resetError();
			this.step = 5;
		},
		back: function () {
			this.resetError();
			this.step = 1;
		},
		register: function () {
			var credentials = {
	          	name: this.parent_name,
	          	email: this.parent_email,
	          	parent_type: this.parent_gender,
	          	password: this.parent_password,
	          	service_id: 2
	        }

	        var that = this;
	        register.signup(this, credentials, function(msg, response) {
	        	that.showAddChild();
	        }, function(msg, response) {
	        	that.logError(msg);
	        })
		},
		addChild: function () {
			this.loading = true;
			var credentials = {
	          	child_name: this.child_name,
	          	child_birthday: this.child_bday_y + '-' + this.child_bday_m + '-' + this.child_bday_d,
	          	child_type: this.child_gender,
	          	user_id: auth.user.get('id')
	        }

	        var that = this;
	        register.addChild(this, credentials, function(msg, response) {
	        	auth.user.addChild(credentials);
                auth.saveUser(auth.user);
	        	that.showAddChildPhoto();
	        }, function(msg, response) {
	        	that.logError(msg);
	        })
		},
		addPhoto: function () {
			

	        var that = this;
	        // auth.addChild(this, credentials, function() {
	        // 	this.resetError();
	        // })
		},
		resetError: function () {
			this.loading = false;
			this.error_message = '';
		},
		logError(msg) {
			this.loading = false;
			var msgStr = '';
			if (typeof msg == 'string') {
				msgStr = msg;
			} else {
				$.each(msg, function(label, value) {
					msgStr += value + ' ';
				})
			}
	        
	        this.error_message = msgStr;
	    },
    }
}
</script>