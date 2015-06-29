# ckeditor-autocomplete-plugin
A Ckeditor Plugin to Show suggestion while you type, the suggestion are defined by you and the action key to trigger plugin is also developer defined.

In order to make a suggestion box, you will have to make your custom plugin to use context menu as suggestion box, please check out the link for the basic knowledge of making ckeditor plugin from here a link

Add this to your config.js, where autocomplete is name of the plugin

config.extraPlugins = 'autocomplete';
Then create a following directory structure/file in the ckeditor folder

ckeditor->plugins->autocomplete->plugin.js
Put the following content in your plugin.js file

CKEDITOR.plugins.add('autocomplete',
            {
                init : function(editor) {

                     var autocompleteCommand = editor.addCommand('autocomplete', {
                        exec : function(editor) {
                            var dummyElement = editor.document
                                    .createElement('span');
                            editor.insertElement(dummyElement);

                            var x = 0;
                            var y = 0;

                            var obj = dummyElement.$;

                            while (obj.offsetParent) {
                                x += obj.offsetLeft;
                                y += obj.offsetTop;
                                obj = obj.offsetParent;
                            }
                            x += obj.offsetLeft;
                            y += obj.offsetTop;

                            dummyElement.remove();

                            editor.contextMenu.show(editor.document
                                    .getBody(), null, x, y);
                        }
                    });
                },

                afterInit : function(editor) {
                    editor.on('key', function(evt) {
                        if (evt.data.keyCode == editor.config.suggestionsTriggerKey.keyCode) {
                            editor.execCommand('autocomplete');
                        }
                    });

                    var firstExecution = true;
                    var dataElement = {};

                     editor.addCommand('reloadSuggetionBox', {
                            exec : function(editor,suggestions) {
                                if (editor.contextMenu) {
                                    dataElement = {};
                                    editor.addMenuGroup('suggestionBoxGroup');

                            $.each(suggestions,function(i, suggestion)
                            {
                                    var suggestionBoxItem = "suggestionBoxItem"+ i; 
                                    dataElement[suggestionBoxItem] = CKEDITOR.TRISTATE_OFF;
                                    editor.addMenuItem(suggestionBoxItem,
                                                                        {
                                        id : suggestion.id,
                                        label : suggestion.label,
                                        group : 'suggestionBoxGroup',
                                        icon  : null,
                                        onClick : function() {
                                            var data = editor.getData();
                                            var selection = editor.getSelection();
                                            var element = selection.getStartElement();
                                            var ranges = selection.getRanges();
                                            ranges[0].setStart(element.getFirst(), 0);
                                            ranges[0].setEnd(element.getFirst(),0);
                                            editor.insertHtml(this.id + '&nbsp;');
                                            },
                                            });
                                    });

                                    if(firstExecution == true)
                                        {
                                            editor.contextMenu.addListener(function(element) {
                                                return dataElement;
                                            });
                                        firstExecution = false;
                                        }
                                }
                            }
                     });

                    delete editor._.menuItems.paste;
                },
            });

Here "suggestions" is the variable passed from a jquery file on your page, the variable holds a list of object having a 'id' and 'label' to be shown in suggestion.

Now in order to configure these suggestions, please perform the following jquery code, after this, whenever '#' is pressed, suggestions will be shown

	$('textarea').ckeditor();
	//Here "CKEDITOR.SHIFT + 51" is the key combination for '#'
	$('textarea#ckeditorBox').ckeditor({ suggestionsTriggerKey: { keyCode: CKEDITOR.SHIFT + 51 }});
		CKEDITOR.on( 'instanceReady', function( evt ) {
			//Here 'Index.suggestions' is the Array which is holding the current list of suggestions
			CKEDITOR.instances.ckeditorBox.execCommand('reloadSuggetionBox',Index.suggestions);
		});

This will load the ckeditor("ckeditor" is name of my ckeditor instance) and configure the plugin to show suggestions currently present int the "Index.suggestions" variable, anytime you need to refresh/change the suggestions you just need to call this function after reloading "Index.suggestions" variable

 	CKEDITOR.instances.ckeditorBox.execCommand('reloadSuggetionBox',Index.suggestions);


