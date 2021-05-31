<script>
import PWAPrompt from "./PWAPrompt";

const deviceCheck = () => {
  const isiOS = /iphone|ipad|ipod/.test(
    window.navigator.userAgent.toLowerCase()
  );
  const isiPadOS =
    navigator.platform === "MacIntel" && navigator.maxTouchPoints > 1;
  const isStandalone =
    "standalone" in window.navigator && window.navigator.standalone;

  return (isiOS || isiPadOS) && !isStandalone;
};

export default {
  components: {
    PWAPrompt,
  },
  props: {
    timesToShow: { type: Number, default: 1 },
    promptOnVisit: { type: Number, default: 1 },
    permanentlyHideOnDismiss: { type: Boolean, default: true },
    copyTitle: {
      type: String,
      default: function () {
        return "Add to Home Screen";
      },
    },
    copyBody: {
      type: String,
      default: function () {
        return "This website has app functionality. Add it to your home screen to use it in fullscreen and while offline.";
      },
    },
    copyShareButtonLabel: {
      type: String,
      default: function () {
        return "1) Press the 'Share' button on the menu bar below.";
      },
    },
    copyAddHomeButtonLabel: {
      type: String,
      default: function () {
        return "2) Press 'Add to Home Screen'.";
      },
    },
    copyClosePrompt: {
      type: String,
      default: function () {
        return "Cancel";
      },
    },
    delay: { type: Number, default: 1000 },
    debug: { type: Boolean, default: false },
    onClose: { type: Function, default: () => {} },
  },

  render: function (h) {
    let promptData = JSON.parse(localStorage.getItem("iosPwaPrompt"));

    if (promptData === null) {
      promptData = { isiOS: deviceCheck(), visits: 0 };
      localStorage.setItem("iosPwaPrompt", JSON.stringify(promptData));
    }

    if (promptData.isiOS || this.debug) {
      const aboveMinVisits = promptData.visits + 1 >= this.promptOnVisit;
      const belowMaxVisits =
        promptData.visits + 1 < this.promptOnVisit + this.timesToShow;

      if (belowMaxVisits || this.debug) {
        localStorage.setItem(
          "iosPwaPrompt",
          JSON.stringify({
            ...promptData,
            visits: promptData.visits + 1,
          })
        );

        if (aboveMinVisits || this.debug) {
          return h(PWAPrompt, {
            props: {
              delay: this.delay,
              copyTitle: this.copyTitle,
              copyBody: this.copyBody,
              copyAddHomeButtonLabel: this.copyAddHomeButtonLabel,
              copyShareButtonLabel: this.copyShareButtonLabel,
              copyClosePrompt: this.copyClosePrompt,
              permanentlyHideOnDismiss: this.permanentlyHideOnDismiss,
              promptData: promptData,
              maxVisits: this.timesToShow + this.promptOnVisit,
              onClose: this.onClose,
            },
          });
        }
      }
    }

    return null;
  },
};
</script>