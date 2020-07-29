<template>
  <section class="container">

    <div
      v-if="current"
      class="fixed fixed--center"
      style="z-index: 3"
      :class="{ 'transition': isVisible }">
      <Vue2InteractDraggable
        v-if="isVisible"
        :interact-out-of-sight-x-coordinate="400"
        :interact-out-of-sight-y-coordinate="200"
        :interact-max-rotation="20"
        :interact-x-threshold="200"
        :interact-y-threshold="200"
        :interact-event-bus-events="interactEventBus"
        @draggedRight="emitAndNext('match')"
        @draggedLeft="emitAndNext('skip')"
        @draggedUp="emitAndNext('favorite')"
        @draggedDown="emitAndNext('reject')"
        class="rounded-borders card card--one">
        <div style="height: 100%">
          <div class="ribbon-wrapper">
            <a id="view-job" :data-id="current.ID" class="ribbon blue" onclick="viewdetail(this)"><i class="material-icons">pageview</i></a>
          </div>
          <div class="user-detail">
            <div class="row justify-content-between">
              <div class="col">
                <h6 class="card-subtitle mb-2">{{ current.job_title }}</h6>
                <small><span class="text-d-grey">{{ current.dated }}</span></small>
                <ul class="list-unstyled mb-2">
                  <li class="media align-items-center mb-1">
                    <i class="material-icons">my_location</i>
                    <h6 class="mt-0 mb-0 font-reg text-d-grey">{{ current.city }}.</h6>
                  </li>
                  <li class="media align-items-center mb-1">
                    <i class="material-icons">money</i>
                    <h6 class="mt-0 mb-0 font-reg text-d-grey">{{ current.pay }}</h6>
                  </li>
                  <li class="media align-items-center mb-1">
                    <i class="material-icons">assignment_ind</i>
                    <h6 class="mt-0 mb-0 font-reg text-d-grey">{{ current.experience }} {{lang_exp}}</h6>
                  </li>
                </ul>
                <h6 class="mb-2">{{lang_skill}}</h6>
                <p v-if="typeof(current.required_skills)!='undefined'" class="skills-list list-inline mb-2">
                  {{ current.required_skills }}
                </p>
              </div>
              <div class="col-auto">
                <img :src="company_logo" class="company_logo img-thumbnail" width="100px"/>
              </div>
            </div>
            <div class="about-summary">
              <h6>{{lang_about}}</h6>
              <p v-html="current.job_description"></p>
            </div>
          </div>
          <div class="text">
            <h4 class="pt-2">{{current.company_name}}</h4>
          </div>
        </div>
      </Vue2InteractDraggable>
    </div>
    <div
      v-if="next"
      class="rounded-borders card card--two fixed fixed--center"
      style="z-index: 2">
      <div style="height: 100%">
        <div class="text">
          <h2>{{next.company_name}}</h2>
        </div>
      </div>
    </div>
    <div class="rounded-borders card card--three fixed fixed--center"
      style="z-index: 1">
      <div style="height: 100% p-5">
        <h3 class=" text-center">{{lang_no}}</h3>
      </div>
    </div>
    <div v-if="current" class="footer fixed">
      <input type="hidden" id="job_ID" v-model="current.ID">
      <div class="btn btn--skip" @click="skip">
          <i class="material-icons">call_missed</i>
      </div>
      <div class="btn btn--decline" @click="reject">
          <i class="material-icons">close</i>
      </div>
      <div class="btn btn--call" @click="openmap">
          <i class="material-icons">my_location</i>
      </div>
      <div class="btn btn--pin" @click="favorite">
          <i class="material-icons">favorite</i>
      </div>
      <div class="btn btn--like" @click="match">
          <i class="material-icons">thumb_up</i>
      </div>
    </div>
  </section>
</template>
<script>
import { Vue2InteractDraggable, InteractEventBus } from 'vue2-interact'
import axios from 'axios'
import { isMobile } from 'mobile-device-detect';
const EVENTS = {
  MATCH: 'match',
  SKIP: 'skip',
  REJECT: 'reject',
  FAVORITE: 'favorite',
}

export default {
  name: 'SwipeableCards',
  components: { Vue2InteractDraggable },
  data() {
    return {
      isVisible: true,
      index: 0,
      skipedindex: 0,
      interactEventBus: {
        draggedRight: EVENTS.MATCH,
        draggedLeft: EVENTS.SKIP,
        draggedUp: EVENTS.FAVORITE,
        draggedDown: EVENTS.REJECT
      },
      cards: [],
      skiped:[],
      skipedRemoveFlag:false,
      baseurl:'',
      lang_about:'',
      lang_skill:'',
      lang_exp:'',
      lang_no:''
    }
  },
  computed: {
    company_logo() {
      var base = this.baseurl+'/../public/uploads/employer/';
      var logo = 'no_logo.jpg';
      if(this.current.company_logo && this.current.company_logo!=''){
        logo = this.current.company_logo;
      }
      return base + logo;
    },
    current() {
      if(this.index < this.cards.length){
        return this.cards[this.index]
      }
      else{
        if(this.skiped.length > 0){
          return this.cards[this.skiped[this.skipedindex % this.skiped.length]]
        }
        else
          return false;
      }
    },
    next() {
      if(this.index < this.cards.length){
        if(this.index+1 == this.cards.length){
          if(this.skiped.length > 0){
            return this.cards[this.skiped[0]];
          }else{
            return false;
          }
        }else{
          return this.cards[this.index + 1]
        }
      }
      else{
        if(this.skiped.length > 0){
          return this.cards[this.skiped[(this.skipedindex+1) % this.skiped.length]]
        }
        else
          return false;
      }
    }
  },
  methods: {
    match() {
      InteractEventBus.$emit(EVENTS.MATCH)
    },
    reject() {
      InteractEventBus.$emit(EVENTS.REJECT)
    },
    favorite() {
      InteractEventBus.$emit(EVENTS.FAVORITE)
    },
    skip() {
      InteractEventBus.$emit(EVENTS.SKIP)
    },
    openmap(){
      if(isMobile){
        if(navigator.platform.indexOf("iPhone") != -1
          || navigator.platform.indexOf("iPad") != -1 
          || navigator.platform.indexOf("iPod") != -1){
          window.location.href=`maps://maps.google.com/maps?addr=${this.current.latitude},${this.current.longitude}&amp;ll=`;
        } else {
          // window.open(`geo:${lat},${lon}`);
          window.open(`https://maps.google.com/maps?daddr=${this.current.latitude},${this.current.longitude}&amp;ll=`);
        }
      } else {
        window.open(`https://www.google.com/maps/@${this.current.latitude},${this.current.longitude},23569m/data=!3m1!1e3?hl=en-US`);
      }
    },
    emitAndNext(event) {
      var job_ID;
      var cardindex;
      if(this.index < this.cards.length){
        cardindex = this.index;
      }else{
        cardindex = this.skiped[this.skipedindex % this.skiped.length]
      }
      this.$emit(event, cardindex)

      job_ID = this.cards[cardindex].ID
      var url = ''
      if(event=='skip'){
        if(this.index < this.cards.length && this.skiped.indexOf(this.index)==-1)
          this.skiped.push(this.index);
      }
      if(event=='reject'){
        if(this.skiped.length>0)
          this.skipedRemoveFlag = true;
        url = 'reject'
      }
      if(event=='favorite'){
        if(this.skiped.length>0)
          this.skipedRemoveFlag = true;
        url = 'wishlist'
      }
      if(event=='match'){
        if(this.skiped.length>0)
          this.skipedRemoveFlag = true;
        url = 'apply'
      }
      if(url!=''){
        axios.post(this.baseurl + '/seeker/job/'+url, 'jobId='+job_ID, { headers: {
          'Content-type': 'application/x-www-form-urlencoded',
          }
        }).then((response) => {
          if(response.data==='0'){
            window.location.href = this.baseurl + '/seeker/my-cv';
          }
        });
      }
      setTimeout(() => this.isVisible = false, 200)
      setTimeout(() => {
        if(this.index < this.cards.length)
          this.index++
        else{
          if(this.skipedRemoveFlag){
            this.skiped.splice(this.skipedindex % this.skiped.length,1);
            this.skipedRemoveFlag = false;
          }else{
            this.skipedindex++
          }
        }
        this.isVisible = true
      }, 200)
    }
  },
  mounted () {
    this.baseurl = document.getElementById('baseurl').value;
    this.lang_about = document.getElementById('lang_about').value;
    this.lang_skill = document.getElementById('lang_skill').value;
    this.lang_no = document.getElementById('lang_no').value;
    this.exp = document.getElementById('lang_exp').value;
    axios.get(this.baseurl + '/employer/api/get_job_data').then((response) => {
      this.cards = response.data;
    });
  }
}
</script>

<style lang="scss" scoped>
.container {
  width: 100%;
  min-height: 700px;
  position:relative;
}


.footer {
  width: 77vw;
  top: 67vh;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  padding-bottom: 30px;
  justify-content: space-around;
  align-items: center;
}

.btn {
  position: relative;
  width: 50px;
  height: 50px;
  padding: .2rem;
  border-radius: 50%;
  background-color: white;
  box-shadow: 0 6px 6px -3px rgba(0,0,0,0.3), 0 10px 14px 1px rgba(0,0,0,0.2), 0 4px 18px 3px rgba(0,0,0,0.12);
  cursor: pointer;
  transition: all .3s ease;
  user-select: none;
  -webkit-tap-highlight-color:transparent;
  &:active {
    transform: translateY(4px);
  }
  i {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    &::before {
      content: '';
    }
  }
  &--call{
    color:#00f;
  }
  &--pin{
    background-color:#03b103;
    color:#fff;
  }
  &--like {
    background-color: red;
    color: white;
    box-shadow: 0 10px 13px -6px rgba(0,0,0,.2), 0 20px 31px 3px rgba(0,0,0,.14), 0 8px 38px 7px rgba(0,0,0,.12);
  }
  &--decline {
    color: red;
  }
  &--skip {
    color: green;
  }
}

.flex {
  display: flex;
  &--center {
    align-items: center;
    justify-content: center;
  }
}

.fixed {
  position: absolute;
  width:100%;
  max-width:600px;
  &--center {
    left: 50%;
    top: 0;
    transform: translate(-50%, 0);
  }
}
.rounded-borders {
  border-radius: 12px;
}
.card {
  background:#fff;
  width: 100%;
  height: 60vh;
  color: white;
  &--one {
    box-shadow: 0 1px 3px rgba(0,0,0,.2), 0 1px 1px rgba(0,0,0,.14), 0 2px 1px -1px rgba(0,0,0,.12);
  }
  &--two {
    transform: translate(-48%, 2%);
    box-shadow: 0 6px 6px -3px rgba(0,0,0,.2), 0 10px 14px 1px rgba(0,0,0,.14), 0 4px 18px 3px rgba(0,0,0,.12);
  }
  &--three {
    transform: translate(-46%, 4%);
    box-shadow: 0 10px 13px -6px rgba(0,0,0,.2), 0 20px 31px 3px rgba(0,0,0,.14), 0 8px 38px 7px rgba(0,0,0,.12);
    color:#333;
  }
  .text {
    position: absolute;
    bottom: 0;
    width: 100%;
    background:#666;
    border-bottom-right-radius: 12px;
    border-bottom-left-radius: 12px;
    text-indent: 20px;
    span {
      font-weight: normal;
    }
  }
}
.media i{
  color: #0b7fff;
  margin-right: 6px;
}
.transition {
  animation: appear 200ms ease-in;
}
.text-center{
  text-align:center;
}
.user-detail{
  color:#111;
  padding:20px;
  height:100%;
  overflow:hidden;
}
.user-icon{
  float:left;
  margin-right:10px;
}
ul{
  padding:0;
}
.media{
  list-style:none;
}
.ribbon-wrapper {
  width: 85px;
  height: 88px;
  overflow: hidden;
  position: absolute;
  top: -3px;
  right: -3px;
}
.ribbon{
  display:block;
  font-size: 14px;
  color: #FFF;
  text-transform: uppercase;
  font-family: 'Montserrat Bold', 'Helvetica Neue', Helvetica, Arial, sans-serif;
  letter-spacing: .05em;
  line-height: 28px;
  text-align: center;
  text-shadow: 0 -1px 0 rgba(0, 0, 0, .4);
  transform: rotate(45deg);
  position: relative;
  padding: 2px 0;
  right: -11px;
  top: 10px;
  width: 100px;
  height: 28px;
  box-shadow: 0 0 3px rgba(0, 0, 0, .3);
  cursor:pointer;
  &.blue {
    background-color: #1a8bbc;
    background-image: linear-gradient(to bottom, #177aa6 45%, #1a8bbc 100%);
    background-repeat: repeat-x;
    :before, :after{
      content: "";
      border-top: 3px solid #115979;
      border-left: 3px solid transparent;
      border-right: 3px solid transparent;
      position: absolute;
      bottom: -3px
    }
    :before{
      left:0;
    }
    :after{
      right:0;
    }
  }
}

@keyframes appear {
  from {
    transform: translate(-48%, 2%);
  }
  to {
    transform: translate(-50%, 0%);
  }
}

</style>
