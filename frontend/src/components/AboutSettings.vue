<script>
import { Avatar, Button, Checkbox, Input, Switch } from "frappe-ui";
import FrappeDriveLogo from "./FrappeDriveLogo.vue";

export default {
  name: "AboutSettings",
  components: { Avatar, Button, Input, Checkbox, Switch, FrappeDriveLogo },
  data() {
    return {
      fullName: this.$store.state.user.fullName,
    };
  },
  computed: {
    isExpanded() {
      return this.$store.state.IsSidebarExpanded;
    },
    firstName() {
      return this.$store.state.user.fullName.split(" ");
    },
    /* fullName() {
      return this.$store.state.user.fullName;
    }, */
    imageURL() {
      return this.$store.state.user.imageURL;
    },
  },
};
</script>
