local Connections = {}
local Objects = {}

--Objects
local Camera = workspace.CurrentCamera

--Services
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

--Functions
local Drawing_new = Drawing.new
local Vector2_new = Vector2.new
local Color3_fromRGB = Color3.fromRGB
local Color3_fromHSV = Color3.fromHSV
local pairs = pairs
local next = next
local wrap = coroutine.wrap

local function IsVector2WithinDrawing(CursorPos, Obj)
	local X1, X2 = Obj.Position.X, Obj.Position.X + Obj.Size.X
	local Y1, Y2 = Obj.Position.Y, Obj.Position.Y + Obj.Size.Y
	return (CursorPos.X >= X1 and CursorPos.X <= X2) and (CursorPos.Y >= Y1 and CursorPos.Y <= Y2)
end

local function NewDrawing(Type, Prop)
	Prop = Prop or {}
	local Obj = Drawing_new(Type)
	for I, V in pairs(Prop) do
		Obj[I] = V
	end
	return Obj
end

--Const
local CursorSuccess, CursorData = pcall(game.HttpGet, game, "https://i.imgur.com/iYx753u.png")
local PickerSuccess, PickerData = pcall(game.HttpGet, game, "https://i.imgur.com/kFz7LMY.png")
local HueSuccess, HueData = pcall(game.HttpGet, game, "https://i.imgur.com/JAYh5cJ.png")
if not CursorSuccess or not PickerSuccess or not HueSuccess then
	error("Failed, Check The Links For the Assets")
end

local Library = {}
local Defaults = {
	MainOutlineColor = Color3_fromRGB(231, 125, 125),
	MainFrameColor = Color3_fromRGB(24, 24, 24),
	TitleColor = Color3_fromRGB(16, 16, 16),
	ButtonColor = Color3_fromRGB(32, 32, 32),
	ButtonEnabledColor = Color3_fromRGB(100, 255, 100),
	MouseColor = Color3_fromRGB(255, 100, 100),
	TextColor = Color3_fromRGB(255, 255, 255),
	Font = Drawing.Fonts.Monospace
}
function Library:CreateWindow(Title, X, Y, TitleHeight)
	local ViewportSize = Camera.ViewportSize
	local Size = Vector2_new(X, Y)
	local Center = Vector2_new(ViewportSize.X / 2 - Size.X / 2, ViewportSize.Y / 2 - Size.Y / 2)
	local Window = {}
	Window.CurrentCategory = nil
	Window.LastCategory = nil
	Window.LastMouseClickOnTitle = Vector2_new()
	Window.Categories = {}
	Window.Visible = true
	Window.BeingDragged = false
	Window.MainOutline = NewDrawing("Square", {
		Visible = true,
		Transparency = 1,
		Size = Size,
		Position = Center,
		Color = Defaults.MainOutlineColor,
		Thickness = 4,
		Filled = false
	})
	Window.TitleBar = NewDrawing("Square", {
		Visible = true,
		Transparency = 1,
		Size = Vector2_new(X, TitleHeight),
		Position = Center,
		Color = Defaults.TitleColor,
		Filled = true
	})
	Window.MainFrame = NewDrawing("Square", {
		Visible = true,
		Filled = true,
		Transparency = 1,
		Size = Vector2_new(X, Y - TitleHeight),
		Position = Vector2_new(Center.X, Center.Y + TitleHeight),
		Color = Defaults.MainFrameColor
	})
	Window.Title = NewDrawing("Text", {
		Visible = true,
		Transparency = 1,
		Color = Defaults.TextColor,
		Text = Title,
		Size = TitleHeight / 2,
		Center = false,
		Outline = true,
		Position = Window.TitleBar.Position + Vector2_new(8, TitleHeight / 4),
		Font = Defaults.Font
	})
	function Window:Move(X2, Y2)
		local MPos = Vector2_new(X2, Y2)
		Window.TitleBar.Position = MPos
		Window.MainOutline.Position = MPos
		Window.MainFrame.Position = Vector2_new(MPos.X, MPos.Y + TitleHeight)
		Window.Title.Position = Window.TitleBar.Position + Vector2_new(8, TitleHeight / 4)
		for i, t in next, Window.Categories do
			t.Toggle.Hitbox.Position = Window.MainOutline.Position + t.Toggle.Offset
			t.Toggle.Text.Position = Window.MainOutline.Position + t.Toggle.TextOffset
			t.Toggle.Line.From = Window.MainOutline.Position + Vector2.new(t.Toggle.Offset.X, TitleHeight * 1.75)
			t.Toggle.Line.To = Window.MainOutline.Position + Vector2.new(t.Toggle.Offset.X + t.Toggle.Hitbox.Size.X, TitleHeight * 1.75)
			for i, t in next, t.Objects do
				if t.Type == "Toggle" or t.Type == "Button" then
					t.Hitbox.Position = Window.MainOutline.Position + t.Offset
					t.Text.Position = Window.MainOutline.Position + t.TextOffset
				end
				if t.Type == "Slider" then
					t.Hitbox.Position = Window.MainOutline.Position + t.Offset
					t.Foreground.Position = Window.MainOutline.Position + t.Offset
					t.Text.Position = Window.MainOutline.Position + t.TextOffset
				end
				if t.Type == "Section" then
					t.Text.Position = Window.MainOutline.Position + t.TextOffset
					t.Line.From = Window.MainOutline.Position + Vector2_new(t.TextOffset.X + t.Text.TextBounds.X + t.Text.Size / 2, t.TextOffset.Y + t.Text.TextBounds.Y / 2)
					t.Line.To = Window.MainOutline.Position + Vector2_new(Window.MainOutline.Size.X - (X * .05), t.TextOffset.Y + t.Text.TextBounds.Y / 2)
				end
				if t.Type == "ComboBox" then
					t.Title.Position = Window.MainOutline.Position + t.TextOffset
					t.Hitbox.Position = Window.MainOutline.Position + t.Offset
					t.Text.Position = Window.MainOutline.Position + t.Offset
					local index = 1
					local DropdownSize = t.Offset.Y
					for i, t in next, t.Buttons do
						DropdownSize = DropdownSize + 20
						index = index + 1
						t.Hitbox.Position = Window.MainOutline.Position + Vector2_new(t.Offset.X, DropdownSize)
						t.Text.Position = Window.MainOutline.Position + Vector2_new(t.TextOffset.X, DropdownSize)
					end
				end
				if t.Type == "TextBox" then
					t.Hitbox.Position = Window.MainOutline.Position + t.Offset
					t.Text.Position = Window.MainOutline.Position + t.Offset
					t.Title.Position = Window.MainOutline.Position + t.TitleOffset
					t.Focused = false
					if t.InputConnection then
						t.InputConnection:Disconnect()
					end
				end
				if t.Type == "ColorPicker" then
					t.Categories.Position = Window.MainOutline.Position + t.CategoryOffset
					t.Picker.Position = Window.MainOutline.Position + t.PickerOffset
					t.Overlay.Position = Window.MainOutline.Position + t.PickerOffset
					t.Hue.Position = Window.MainOutline.Position + t.HueOffset
					for i, o in next, t.Items do
						o.BG.Position = Window.MainOutline.Position + (t.CategoryOffset + Vector2_new(0, o.MyOffset))
						o.Text.Position = Window.MainOutline.Position + (t.CategoryOffset + Vector2_new(0, o.MyOffset))
						o.ColorBG.Position = Window.MainOutline.Position + Vector2_new(t.CategoryOffset.x + (t.Categories.Size.x * .75), t.CategoryOffset.Y + o.MyOffset + 2)
					end
				end
			end
		end
	end
	function Window:ChangeCategory(Category)
		Window.LastCategory = Window.CurrentCategory
		Window.CurrentCategory = Category
		for i, t in next, Window.Categories do
			t.Toggle.Line.Visible = t == Category
			for i2, o in next, t.Objects do
				if o.Type == "Toggle" or o.Type == "Button" then
					o.Hitbox.Visible = t == Category
					o.Text.Visible = t == Category
				end
				if o.Type == "Slider" then
					o.Hitbox.Visible = t == Category
					o.Foreground.Visible = t == Category
					o.Text.Visible = t == Category
				end
				if o.Type == "Section" then
					o.Text.Visible = t == Category
					o.Line.Visible = t == Category
				end
				if o.Type == "ComboBox" then
					o.Hitbox.Visible = t == Category
					o.Text.Visible = t == Category
					o.Title.Visible = t == Category
					if o.Open and t ~= Category then
						o:Toggle()
					end
				end
				if o.Type == "TextBox" then
					o.Hitbox.Visible = t == Category
					o.Text.Visible = t == Category
					o.Title.Visible = t == Category
					o.Focused = false
					if t.InputConnection then
						t.InputConnection:Disconnect()
					end
				end
				if o.Type == "ColorPicker" then
					o.Categories.Visible = t == Category
					o.Picker.Visible = t == Category
					o.Overlay.Visible = t == Category
					o.Hue.Visible = t == Category
					for i3, o2 in next, o.Items do
						o2.BG.Visible = t == Category
						o2.Text.Visible = t == Category
						o2.ColorBG.Visible = t == Category
					end
				end
			end
		end
	end
	function Window:Toggle()
		Window.Visible = not Window.Visible
		Window.MainOutline.Visible = not Window.MainOutline.Visible
		Window.TitleBar.Visible = not Window.TitleBar.Visible
		Window.Title.Visible = not Window.Title.Visible
		Window.MainFrame.Visible = not Window.MainFrame.Visible
		Window.Mouse.Visible = not Window.Mouse.Visible
		Window.BeingDragged = false
		for i, t in next, Window.Categories do
			t.Toggle.Hitbox.Visible = Window.MainOutline.Visible
			t.Toggle.Text.Visible = Window.MainOutline.Visible
		end
        -- TODO: Add blur again without using blur effect
		if Window.Visible then
			Window:ChangeCategory(Window.LastCategory)
		else
			Window:ChangeCategory(nil)
			if Window.CurrentCategory ~= nil then
				for i, t in next, Window.CurrentCategory.Objects do
					if t.Type == "Textbox" then
						t.Focused = false
						if t.InputConnection then
							t.InputConnection:Disconnect()
						end
					end
				end
			end
		end
	end
	function Window:Init()
		for i, c in next, Window.Categories do
			for i, o in next, c.Objects do
				if o.Type == "ComboBox" then
					o:Init()
				end
			end
		end
		Window.Mouse = NewDrawing("Image", {
			Visible = true,
			Transparency = 1,
			Size = Vector2_new(36, 36),
			Position = UIS:GetMouseLocation(),
			Data = CursorData
		})
	end
	function Window:CreateCategory(name)
		local Category = {}
		Category.UISize = 0
		Category.Name = name
		Category.Toggle = {
			Type = "Toggle",
			Offset = Vector2_new((X / 5) * #Window.Categories, TitleHeight),
			TextOffset = Vector2_new((X / 5) * #Window.Categories + X / 10, TitleHeight * 1.15),
			Hitbox = NewDrawing("Square", {
				Visible = true,
				Transparency = 1,
				Filled = true,
				Size = Vector2_new(X / 5, TitleHeight * .75),
				Position = Window.MainOutline.Position + Vector2_new((X / 5) * #Window.Categories, TitleHeight),
				Color = Defaults.ButtonColor
			}),
			Text = NewDrawing("Text", {
				Visible = true,
				Text = name,
				Position = Window.MainOutline.Position + Vector2_new((X / 5) * #Window.Categories + X / 10, TitleHeight * 1.15),
				Size = TitleHeight * .5,
				Center = true,
				Color = Defaults.TextColor,
				Font = Defaults.Font,
				Outline = true
			}),
			Line = NewDrawing("Line", {
				Visible = false,
				Transparency = 1,
				Thickness = 1,
				Color = Defaults.MainOutlineColor,
				From = Window.MainOutline.Position + Vector2.new((X / 5) * #Window.Categories, TitleHeight * 1.75),
				To = Window.MainOutline.Position + Vector2.new(((X / 5) * #Window.Categories) + X / 5, TitleHeight * 1.75)
			})
		}
		Category.Objects = {}
		function Category:CreateSection(name)
			local Section = {
				Type = "Section",
				TextOffset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				Text = NewDrawing("Text", {
					Visible = true,
					Text = name,
					Font = Defaults.Font,
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
					Size = 18,
					Color = Defaults.TextColor,
					Outline = true
				}),
				Callback = callback,
				State = state
			}
			Section.Line = NewDrawing("Line", {
				Visible = true,
				Color = Defaults.MainOutlineColor,
				From = Window.MainOutline.Position + Vector2_new(Section.TextOffset.X + Section.Text.TextBounds.X + Section.Text.Size / 2, Section.TextOffset.Y + Section.Text.TextBounds.Y / 2),
				To = Window.MainOutline.Position + Vector2_new(Window.MainOutline.Size.X - (X * .05), Section.TextOffset.Y + Section.Text.TextBounds.Y / 2)
			})
			Category.UISize = Category.UISize + 24
			Category.Objects[#Category.Objects + 1] = Section
			return Toggle
		end
		function Category:CreateToggle(name, state, callback)
			local Toggle = {
				Type = "Toggle",
				Offset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				TextOffset = Vector2_new((X * .05) + 32, (TitleHeight * 2.3) + Category.UISize),
				Hitbox = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Filled = true,
					Size = Vector2_new(16, 16),
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize), -- This is stupid and bad there is a reason i am failing math fuck my life A
					Color = Defaults.ButtonColor
				}),
				Text = NewDrawing("Text", {
					Visible = true,
					Text = name,
					Font = Defaults.Font,
					Position = Window.MainOutline.Position + Vector2_new((X * .05) + 32, (TitleHeight * 2.3) + Category.UISize),
					Size = 16,
					Color = Defaults.TextColor,
					Outline = true
				}),
				Callback = callback,
				State = false
			}
			function Toggle.Toggle()
				Toggle.State = not Toggle.State
				callback(Toggle.State)
				Toggle.Hitbox.Color = Toggle.State and Defaults.MainOutlineColor or Defaults.ButtonColor
			end
			if Toggle.State ~= state then
				Toggle.Toggle()
			end
			Category.UISize = Category.UISize + 24
			Category.Objects[#Category.Objects + 1] = Toggle
			return Toggle
		end
		function Category:CreateSlider(name, minimum, maximum, state, callback)
			local Slider = {
				Type = "Slider",
				_min = minimum,
				_max = maximum,
				Amount = state,
				Offset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 24),
				TextOffset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				Hitbox = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Filled = true,
					Size = Vector2_new((X * .9), 16),
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 24), -- This is stupid and bad there is a reason i am failing math fuck my life A
					Color = Defaults.ButtonColor
				}),
				Foreground = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Filled = true,
					Size = Vector2_new((X * .9) * (state / maximum), 16),
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 24),
					Color = Defaults.MainOutlineColor
				}),
				Text = NewDrawing("Text", {
					Visible = true,
					Text = name .. ": " .. state,
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
					Size = 16,
					Color = Defaults.TextColor,
					Font = Defaults.Font,
					Outline = true
				}),
				Callback = callback,
				State = state
			}
			Category.UISize = Category.UISize + 48
			function Slider.Slid(Amount)
				Slider.Amount = Amount
				Slider.Text.Text = name .. ": " .. Amount
				callback(Amount)
			end
			Slider.Slid(Slider.Foreground.Size.X)
			Category.Objects[#Category.Objects + 1] = Slider
			return Slider
		end
		function Category:CreateButton(name, callback)
			local Button = {
				Type = "Button",
				MaxValue = max_value,
				Offset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				TextOffset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				Hitbox = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Filled = true,
					Size = Vector2_new(X * .2, 20),
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize), -- This is stupid and bad there is a reason i am failing math fuck my life A
					Color = Defaults.ButtonColor
				}),
				Text = NewDrawing("Text", {
					Visible = true,
					Text = name,
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
					Size = 20,
					Color = Defaults.TextColor,
					Font = Defaults.Font,
					Outline = true
				}),
				Callback = callback,
				State = state
			}
			Category.UISize = Category.UISize + 24
			Button.Hitbox.Size = Button.Text.TextBounds
			Category.Objects[#Category.Objects + 1] = Button
			return Button
		end
		function Category:CreateTextBox(name, callback)
			local TextBox = {
				Type = "TextBox",
				Focused = false,
				Offset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				TitleOffset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				Hitbox = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Filled = true,
					Size = Vector2_new(X * .65, 20),
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize), -- This is stupid and bad there is a reason i am failing math fuck my life A
					Color = Defaults.ButtonColor
				}),
				Title = NewDrawing("Text", {
					Visible = true,
					Text = name,
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
					Size = 20,
					Color = Defaults.TextColor,
					Font = Defaults.Font,
					Outline = true
				}),
				Text = NewDrawing("Text", {
					Visible = true,
					Text = "",
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
					Size = 20,
					Color = Defaults.TextColor,
					Font = Defaults.Font,
					Outline = true
				}),
				Callback = callback,
				State = state
			}
			TextBox.Title.Position = TextBox.Title.Position + Vector2_new(TextBox.Hitbox.Size.X + (X * .025))
			TextBox.TitleOffset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize) + Vector2_new(TextBox.Hitbox.Size.X + (X * .025))
			Category.UISize = Category.UISize + 24
			Category.Objects[#Category.Objects + 1] = TextBox
			return Button
		end
		function Category:CreateColorPicker(Items, Callback)
			local ColorPicker = {
				Type = "ColorPicker",
				SelectedItem = "",
				Callback = Callback,
				HueDragged = false,
				PickerDragged = false,
				CategoryOffset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
				PickerOffset = Vector2_new((X * .05) + X * .31, (TitleHeight * 2.3) + Category.UISize),
				HueOffset = Vector2_new((X * .05) + X * .31, ((TitleHeight * 2.3) + Category.UISize) + ((X * .59) - 46)),
				Items = {},
				ItemOffset = 0,
				Categories = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize),
					Size = Vector2.new(X * .3, (X * .59) - 30),
					Filled = true,
					Color = Defaults.TitleColor
				}),
				Picker = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Position = Window.MainOutline.Position + Vector2_new((X * .05) + X * .31, (TitleHeight * 2.3) + Category.UISize),
					Size = Vector2.new(X * .59, (X * .59) - 48),
					Filled = true,
					Color = Color3_fromHSV(.5, 1, 1)
				}),
				Overlay = NewDrawing("Image", {
					Visible = true,
					Transparency = 1,
					Position = Window.MainOutline.Position + Vector2_new((X * .05) + X * .31, (TitleHeight * 2.3) + Category.UISize),
					Data = PickerData,
					Size = Vector2.new(X * .59, (X * .59) - 48)
				}),
				Hue = NewDrawing("Image", {
					Visible = true,
					Transparency = 1,
					Position = Window.MainOutline.Position + Vector2_new((X * .05) + X * .31, ((TitleHeight * 2.3) + Category.UISize) + ((X * .59) - 46)),
					Data = HueData,
					Size = Vector2.new(X * .59, 16)
				})
			}
			function ColorPicker:ForceColor(c)
				if ColorPicker.Items[ColorPicker.SelectedItem] ~= nil then
					local h = c:ToHSV()
					ColorPicker.Picker.Color = Color3_fromHSV(h, 1, 1)
					ColorPicker.Items[ColorPicker.SelectedItem].ColorBG.Color = c
				end
			end
			local first_item = true
			for i, t in next, Items do
				local category = {
					MyOffset = ColorPicker.ItemOffset,
					Color = t,
					BG = NewDrawing("Square", {
						Visible = true,
						Transparency = first_item and 1 or 0,
						Filled = true,
						Position = Window.MainOutline.Position + Vector2_new(ColorPicker.CategoryOffset.x, ColorPicker.CategoryOffset.Y + ColorPicker.ItemOffset),
						Size = Vector2.new(ColorPicker.Categories.Size.x, 20),
						Color = Defaults.ButtonColor
					}),
					Text = NewDrawing("Text", {
						Visible = true,
						Text = tostring(i),
						Transparency = 1,
						Position = Window.MainOutline.Position + Vector2_new(ColorPicker.CategoryOffset.x, ColorPicker.CategoryOffset.Y + ColorPicker.ItemOffset),
						Size = 20,
						Color = Defaults.TextColor,
						Font = Defaults.Font,
						Outline = true
					}),
					ColorBG = NewDrawing("Square", {
						Visible = true,
						Transparency = 1,
						Filled = true,
						Position = Window.MainOutline.Position + Vector2_new(ColorPicker.CategoryOffset.x + (ColorPicker.Categories.Size.x * .75), ColorPicker.CategoryOffset.Y + ColorPicker.ItemOffset + 2),
						Size = Vector2.new(ColorPicker.Categories.Size.x * .2, 16),
						Color = t
					})
				}
				ColorPicker.ItemOffset = ColorPicker.ItemOffset + 20
				ColorPicker.Items[i] = category
				if first_item then
					ColorPicker.SelectedItem = i
				end
				first_item = false
			end
			Category.UISize = Category.UISize + ColorPicker.CategoryOffset.y + 24
			Category.Objects[#Category.Objects + 1] = ColorPicker
			return ColorPicker
		end
		function Category:CreateComboBox(name, callback, things, index)
			local ComboBox = {
				Type = "ComboBox",
				MaxValue = max_value,
				Item = things[1],
				Open = false,
				DropdownSize = (TitleHeight * 2.3) + Category.UISize + 24,
				Offset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 24),
				TextOffset = Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 4),
				Callback = callback,
				Title = NewDrawing("Text", {
					Visible = true,
					Transparency = 1,
					Text = name,
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 4),
					Size = 16,
					Color = Defaults.TextColor,
					Font = Defaults.Font,
					Outline = true
				}),
				Hitbox = NewDrawing("Square", {
					Visible = true,
					Transparency = 1,
					Filled = true,
					Size = Vector2_new(X * .4, 20),
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 24), -- This is stupid and bad there is a reason i am failing math fuck my life A
					Color = Defaults.ButtonColor
				}),
				Text = NewDrawing("Text", {
					Visible = true,
					Text = things[1],
					Position = Window.MainOutline.Position + Vector2_new((X * .05), (TitleHeight * 2.3) + Category.UISize + 24),
					Size = 20,
					Color = Defaults.TextColor,
					Font = Defaults.Font,
					Outline = true
				}),
				Buttons = {},
				State = state
			}
			for i, t in next, things do
				if t == index then
					ComboBox.Item = t
					ComboBox.Text.Text = t
				end
			end
			function ComboBox:Init()
				for i, t in next, things do
					ComboBox.DropdownSize = ComboBox.DropdownSize + 20
					local Button = {
						Type = "Button",
						MaxValue = max_value,
						Offset = Vector2_new((X * .05), ComboBox.DropdownSize),
						TextOffset = Vector2_new((X * .05), ComboBox.DropdownSize),
						Callback = callback,
						Hitbox = NewDrawing("Square", {
							Visible = false,
							Transparency = 1,
							Filled = true,
							Size = Vector2_new(X * .4, 20),
							Position = Window.MainOutline.Position + Vector2_new((X * .05), ComboBox.DropdownSize), -- This is stupid and bad there is a reason i am failing math fuck my life A
							Color = Defaults.ButtonColor
						}),
						Text = NewDrawing("Text", {
							Visible = false,
							Text = t,
							Position = Window.MainOutline.Position + Vector2_new((X * .05), ComboBox.DropdownSize),
							Size = 20,
							Color = Defaults.TextColor,
							Font = Defaults.Font,
							Outline = true
						}),
						State = state
					}
					ComboBox.Buttons[t] = Button
				end
			end
			function ComboBox:Toggle()
				ComboBox.Open = not ComboBox.Open
				for i, t in next, ComboBox.Buttons do
					t.Hitbox.Visible = ComboBox.Open
					t.Text.Visible = ComboBox.Open
				end
			end
			Category.UISize = Category.UISize + 48
			Category.Objects[#Category.Objects + 1] = ComboBox
			return ComboBox
		end
		Window.Categories[#Window.Categories + 1] = Category
		Index = #Window.Categories
		return Category
	end
	Connections.InputBegan = UIS.InputBegan:Connect(function(_)
		if _.UserInputType == Enum.UserInputType.MouseButton1 and IsVector2WithinDrawing(UIS:GetMouseLocation(), Window.TitleBar) and Window.Visible then
			Window.BeingDragged = true
			Window.LastMouseClickOnTitle = UIS:GetMouseLocation()
		end
		if _.UserInputType == Enum.UserInputType.MouseButton1 then
			for i, t in next, Window.Categories do
				if Window.Visible and IsVector2WithinDrawing(UIS:GetMouseLocation(), t.Toggle.Hitbox) then
					Window:ChangeCategory(t)
					break
				end
			end
			if Window.CurrentCategory then
				for i, o in next, Window.CurrentCategory.Objects do
					if o.Type == "ComboBox" and o.Open then
						local finished = false
						for i, b in next, o.Buttons do
							if IsVector2WithinDrawing(UIS:GetMouseLocation(), b.Hitbox) then
								o.Item = b.Text.Text
								o.Text.Text = o.Item
								o:Toggle()
								wrap(o.Callback)(o.Item)
								finished = true
							end
						end
						if finished then
							break
						end
					end
					if (o.Type == "Toggle" or o.Type == "ComboBox") and IsVector2WithinDrawing(UIS:GetMouseLocation(), o.Hitbox) then
						o:Toggle()
						break
					end
					if o.Type == "Button" and IsVector2WithinDrawing(UIS:GetMouseLocation(), o.Hitbox) then
						o.Callback()
					end
					if o.Type == "Slider" and IsVector2WithinDrawing(UIS:GetMouseLocation(), o.Hitbox) then
						o.BeingDragged = true
						RunService:BindToRenderStep("DragSlider", 1, function()
							if o.BeingDragged then
								local newval = (Mouse.X - o.Hitbox.Position.X) / o.Hitbox.Size.X
								newval = math.clamp(newval, 0, 1)
								local v = math.floor(o._min + (o._max - o._min) * newval)
								v = math.floor(v * 10) / 10
								o.Foreground.Size = Vector2_new(newval * o.Hitbox.Size.X, o.Hitbox.Size.Y)
								o.Slid(v)
							else
								RunService:UnbindFromRenderStep("DragSlider")
							end
						end)
					end
					if o.Type == "ColorPicker" then
						if IsVector2WithinDrawing(UIS:GetMouseLocation(), o.Hue) then
							o.HueDragged = true
							RunService:BindToRenderStep("DragHue", 1, function()
								if o.HueDragged then
									local percent = (Mouse.X - o.Hue.Position.X) / o.Hue.Size.X
									percent = math.clamp(percent, 0, 1)
									o.Picker.Color = Color3_fromHSV(percent, 1, 1)
									if o.Items[o.SelectedItem] ~= nil then
										local _, s, v = o.Items[o.SelectedItem].Color:ToHSV()
										o.Items[o.SelectedItem].ColorBG.Color = Color3_fromHSV(percent, s, v)
										o.Items[o.SelectedItem].Color = Color3_fromHSV(percent, s, v)
										o.Callback(o.Items[o.SelectedItem].Color)
									end
								else
									RunService:UnbindFromRenderStep("DragHue")
								end
							end)
							break
						end
						if IsVector2WithinDrawing(UIS:GetMouseLocation(), o.Picker) then
							o.PickerDragged = true
							RunService:BindToRenderStep("DragPicker", 1, function()
								if o.PickerDragged then
									local MouseLocation = UIS:GetMouseLocation()
									local percentx = (MouseLocation.X - o.Picker.Position.X) / o.Picker.Size.X
									local percenty = ((o.Picker.Position.Y + o.Picker.Size.Y) - MouseLocation.Y) / o.Picker.Size.Y
									percentx = math.clamp(percentx, 0, 1)
									percenty = math.clamp(percenty, 0, 1)
									if o.Items[o.SelectedItem] ~= nil then
										local h = o.Picker.Color:ToHSV()
										o.Items[o.SelectedItem].ColorBG.Color = Color3_fromHSV(h, percentx, percenty)
										o.Items[o.SelectedItem].Color = Color3_fromHSV(h, percentx, percenty)
										o.Callback(o.Items[o.SelectedItem].Color)
									end
								else
									RunService:UnbindFromRenderStep("DragPicker")
								end
							end)
							break
						end
						for i2, o2 in next, o.Items do
							if IsVector2WithinDrawing(UIS:GetMouseLocation(), o2.BG) then
								o.SelectedItem = i2
								o2.BG.Transparency = 1 -- GET OUT OF MY HEAD  GET OUT OF MY HEAD  GET OUT OF MY HEAD  GET OUT OF MY HEAD  GET OUT OF MY HEAD  GET OUT OF MY HEAD  GET OUT OF MY HEAD  GET OUT OF MY HEAD  GET OUT OF MY HEAD
								for i3, o3 in next, o.Items do
									if o3 ~= o2 then
										o3.BG.Transparency = 0
									end
								end
								break
							end
						end
					end
					if o.Type == "TextBox" and IsVector2WithinDrawing(UIS:GetMouseLocation(), o.Hitbox) then
						o.Focused = true
						o.InputConnection = UIS.InputBegan:Connect(function(_, GP)
							if _.UserInputType == Enum.UserInputType.Keyboard then
								if _.KeyCode == Enum.KeyCode.Return then
									o.Callback(o.Text.Text)
									o.Focused = false
									o.InputConnection:Disconnect()
								end
								if _.KeyCode == Enum.KeyCode.Space then
									o.Text.Text = o.Text.Text .. " "
								end
								if _.KeyCode == Enum.KeyCode.Backspace then
									o.Text.Text = string.sub(o.Text.Text, 1, #o.Text.Text - 1)
								end
								if _.KeyCode ~= Enum.KeyCode.Backspace and #(tostring(_.KeyCode)) == 14 then
									o.Text.Text = o.Text.Text .. string.sub(tostring(_.KeyCode), 14, 14)
								end
							end
						end)
					elseif o.Type == "TextBox" and o.Focused then
						o.Callback(o.Text.Text)
						o.InputConnection:Disconnect()
						o.Focused = false
					end
				end
			end
		end
		if _.UserInputType == Enum.UserInputType.Keyboard and _.KeyCode == Enum.KeyCode.LeftAlt then
			Window:Toggle()
		end
	end)
	Connections.InputEnded = UIS.InputEnded:Connect(function(_)
		if _.UserInputType == Enum.UserInputType.MouseButton1 then
			Window.BeingDragged = false
			if Window.CurrentCategory then
				for i, o in next, Window.CurrentCategory.Objects do
					if o.Type == "Slider" then
						o.BeingDragged = false
					end
					if o.Type == "ColorPicker" then
						o.HueDragged = false
						o.PickerDragged = false
					end
				end
			end
		end
	end)
	Connections.OnMouseMove = UIS.InputChanged:Connect(function(_)
		if Window.Mouse and _.UserInputType == Enum.UserInputType.MouseMovement then
			Window.Mouse.Visible = (IsVector2WithinDrawing(UIS:GetMouseLocation(), Window.MainFrame) or IsVector2WithinDrawing(UIS:GetMouseLocation(), Window.TitleBar)) and Window.Visible
			Window.Mouse.Position = UIS:GetMouseLocation() - Window.Mouse.Size / 2
			if Window.BeingDragged then
				local Delta = (UIS:GetMouseLocation() - Window.LastMouseClickOnTitle)
				Window.LastMouseClickOnTitle = UIS:GetMouseLocation()
				Window:Move(Window.MainOutline.Position.X + Delta.X, Window.MainOutline.Position.Y + Delta.Y)
			end
		end
	end)
	Objects.Window = Window
	return Window
end

return Library
