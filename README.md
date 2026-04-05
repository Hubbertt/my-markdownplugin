{
	"FileVersion": 3,
	"Version": 1,
	"VersionName": "1.0.0",
	"FriendlyName": "AdvancedMarkdownFlow",
	"Description": "AdvancedMarkdownFlow: standalone Markdown/GFM display plugin for Unreal Engine with optional provider integrations and a clean runtime/editor boundary.",
	"Category": "UI",
	"CreatedBy": "Hubbertt",
	"CreatedByURL": "https://github.com/Hubbertt",
	"DocsURL": "https://github.com/Hubbertt",
	"MarketplaceURL": "",
	"SupportURL": "mailto:hubertwei97@gmail.com",
	"EngineVersion": "5.7.0",
	"CanContainContent": true,
	"Installed": true,
	"Modules": [
		{
			"Name": "MarkdownFlow",
			"Type": "Runtime",
			"LoadingPhase": "Default",
			"PlatformAllowList": [
				"Win64",
				"Mac",
				"IOS",
				"Android"
			]
		},
		{
			"Name": "MarkdownFlowEditor",
			"Type": "Editor",
			"LoadingPhase": "Default"
		},
		{
			"Name": "MarkdownFlowEditorTests",
			"Type": "Editor",
			"LoadingPhase": "Default"
		}
	],
	"Plugins": [
		{
			"Name": "AdvancedLatexRenderer",
			"Enabled": true,
			"Optional": true
		}
	]
}
