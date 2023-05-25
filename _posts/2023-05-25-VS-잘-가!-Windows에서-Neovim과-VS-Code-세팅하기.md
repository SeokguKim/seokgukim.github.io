---
title: VS 잘 가! Windows에서 Neovim과 VS Code 세팅하기
layout: post
tags: [blog, dev, nvim, cpp]
---
### Visual Studio는 더럽다
내가 뭘 모를 시절엔 그냥 Microsoft의 Viusal Studio가 최고인줄 알았다.  
... 아니었지만.

계속 CP와 기타 개발을 하다보니, VS의 답답한 로딩속도와 엄청난 용량은 부담이 되기 시작했고,  
환경 자체를 이사할 필요성에 대해 깨닫게 되었다.  

그래서 VS Code와 그 확장, Neovim 에 해당하는 더 가벼운 텍스트 에디터로 개발환경을 이사하게 되었는데...  
처음 하는 거다 보니까 시행착오가 많았다.

이번엔 그 시행착오들에 대해 써 보고자 한다.
### 일단 Windows라는게 문제다
우선 내 노트북은 개발 외에도 게임, 문서작업 등 범용적인 목적을 생각하고 Windows가 포함된 제품을 구매한 것이다.

살 때는 별로 상관이 없었지만, 개발을 더 깊게 파고들면 파고들수록 Linux 등에서는 기본적으로 있는 바이너리가 없다거나...  
개발을 하려면 꼬운 상황들이 많이 벌어지는 중이다.  
~~(WSL을 깔던가 해야지.)~~

그렇기에, 이 경우 시도할 수 있는 해결책으로 MSYS를 이용해 MinGW와 Clangd를 설치하기로 했다.

### MSYS 설치
VS Code에서 GNU 도구들을 사용하기 위한 내용을 검색해 봤을 때, 가장 유력한 것은 MSYS를 이용하는 것.  
[MSYS](https://www.msys2.org/)를 시스템에 설치한 후, pacman 커맨드를 통해 64비트 버전의 MinGW 툴체인을 설치했다.

```sh
pacman -S --needed base-devel mingw-w64-x86_64-toolchain
```

이렇게 한 경우엔 Clangd도 MSYS를 통해 설치되어야 정상적으로 돌아갔다.  
~~(LLVM을 다른 경로로 직접 설치하는 경우 MSYS를 통해 설치된 MinGW를 못 찾았다)~~

```sh
pacman -S mingw-w64-x86_64-clang-15
```

패키지를 받았다면 잊지 않고 시스템 환경 변수 편집을 통해 MinGW와 Clangd를 시스템 경로에 추가하자.  
이렇게 MinGW와 Clangd가 MSYS를 통해 설치되었다면, 기본은 된 셈.

### VS Code 세팅
VS Code는 다행히도 GUI 기반이다보니 몇 번의 클릭 정도로 해결이 되었다.  
몇 개의 확장만 깔아주니 알아서 설정을 잡아주는 유능한 친구.

<img width="182" alt="image" src="https://github.com/SeokguKim/seokgukim.github.io/assets/43718966/a95edc59-a9b1-4f78-afe2-44ec36d0cb19">

대충 C/C++ 도구와 CMake, Git 기능에 해당하는 확장들을 설치하니 디버그 설정까지 알아서 만들어줬다.  
역시 아직 GUI 기반 툴이 내겐 편한 듯.

<img width="1075" alt="image" src="https://github.com/SeokguKim/seokgukim.github.io/assets/43718966/ba175d86-f62d-492f-b0b3-5028456b9748">

### Neovim 세팅
터미널 기반의 에디터인 Neovim으로도 설정을 시도해 보았다.  
일단 Neovim을 공식 리포지토리로부터 받아서 설치하고, **lazy.nvim**을 플러그인 매니저로 세팅했다.

대충 닷파일 설정은 이렇다.

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({
	--Theme
	"EdenEast/nightfox.nvim",
	--Status
	"nvim-tree/nvim-web-devicons",
	"bluz71/nvim-linefly",
	--cp helper
	"MunifTanjim/nui.nvim",
	"xeluxee/competitest.nvim",
	--Emmet
	"mattn/emmet-vim",
	--Git
	"tpope/vim-fugitive",
	"airblade/vim-gitgutter",
	--DAP
	"mfussenegger/nvim-dap",
	"rcarriga/nvim-dap-ui",
	"theHamsta/nvim-dap-virtual-text",
	--TreeSitter
	"nvim-treesitter/nvim-treesitter",
	--Telescope
	{
    'nvim-telescope/telescope.nvim', tag = '0.1.1',
      dependencies = { 'nvim-lua/plenary.nvim' }
    },
	--Auto completion
	"neovim/nvim-lspconfig",
	 { "hrsh7th/nvim-cmp",
		dependencies = {
			"hrsh7th/cmp-nvim-lsp",
			"hrsh7th/cmp-buffer",
			"hrsh7th/cmp-path",
			"hrsh7th/cmp-cmdline",
			"hrsh7th/cmp-calc",
			"hrsh7th/cmp-nvim-lsp-signature-help",
			"hrsh7th/cmp-nvim-lsp-document-symbol",
			"hrsh7th/cmp-nvim-lua",
			"dcampos/nvim-snippy"
		},
	},
	--TabLine
	{'romgrk/barbar.nvim',
		dependencies = {
		'lewis6991/gitsigns.nvim', -- OPTIONAL: for git status
		},
		init = function() vim.g.barbar_auto_setup = false end,
		version = '^1.0.0', -- optional: only update when a new 1.x version is released
	},
})

--TreeSitter
require "nvim-treesitter.install".prefer_git = false
require "nvim-treesitter.install".compilers = { "clang", "gcc" }
require"nvim-treesitter.configs".setup {
  highlight = {
    enable = true,
    -- Setting this to true will run `:h syntax` and tree-sitter at the same time.
    -- Set this to `true` if you depend on "syntax" being enabled (like for indentation).
    -- Using this option may slow down your editor, and you may see some duplicate highlights.
    -- Instead of true it can also be a list of languages
    additional_vim_regex_highlighting = false,
  },
}

--fzf Ag
vim.g.ackprg = "ag --vimgrep"

--NvimTree and Status Line

vim.opt.termguicolors = true

require("nvim-web-devicons").setup()
--CP helper
require("competitest").setup{
	template_file = {
		cpp = "C:/Users/rokja/ps/template/basic.cpp",
	}
}

--DAP Config
require("dapconfig").setup()
require("dapui").setup()
require("nvim-dap-virtual-text").setup()

vim.fn.sign_define("DapBreakpoint",{ text ="🔴", texthl ="LspDiagnosticsSignError", linehl ="", numhl =""})
vim.fn.sign_define("DapStopped",{ text ="▶️", texthl ="LspDiagnosticsSignInformation", linehl ="DiagnosticUnderlineInfo", numhl ="LspDiagnosticsSignInformation"})
vim.fn.sign_define("DapBreakpointRejected",{ text ="🚫", texthl ="LspDiagnosticsSignHint", linehl ="", numhl =""})

vim.g.dap_virtual_text = true

--AutoCompletion
local cmp = require"cmp"

cmp.setup({
    snippet = {
      -- REQUIRED - you must specify a snippet engine
		expand = function(args)
			require("snippy").expand_snippet(args.body) -- For `snippy` users.
		end,
    },
    window = {
		completion = cmp.config.window.bordered(),
		documentation = cmp.config.window.bordered(),
    },
    mapping = cmp.mapping.preset.insert({
		["<C-b>"] = cmp.mapping.scroll_docs(-4),
		["<C-f>"] = cmp.mapping.scroll_docs(4),
		["<C-Space>"] = cmp.mapping.complete(),
		["<C-e>"] = cmp.mapping.abort(),
		["<CR>"] = cmp.mapping.confirm({ select = true }), -- Accept currently selected item. Set `select` to `false` to only confirm explicitly selected items.
    }),
    sources = cmp.config.sources({
		{ name = "nvim_lsp" },
		{ name = "snippy" }, -- For snippy users.
    }, 
	{
		{ name = "buffer" },
    })
})

-- Set configuration for specific filetype.
cmp.setup.filetype("gitcommit", {
    sources = cmp.config.sources({
		{ name = "cmp_git" }, -- You can specify the `cmp_git` source if you were installed it.
	}, 
	{
		{ name = "buffer" },
	})
 })

-- Use buffer source for `/` and `?` (if you enabled `native_menu`, this won"t work anymore).
cmp.setup.cmdline({ "/", "?" }, {
	mapping = cmp.mapping.preset.cmdline(),
	sources = {
		{ name = "buffer" }
    }
})

-- Use cmdline & path source for ":" (if you enabled `native_menu`, this won"t work anymore).
cmp.setup.cmdline(":", {
    mapping = cmp.mapping.preset.cmdline(),
    sources = cmp.config.sources({
      { name = "path" }
    }, {
      { name = "cmdline" }
    })
})

  -- Set up lspconfig.
local capabilities = require("cmp_nvim_lsp").default_capabilities()
  -- Replace <YOUR_LSP_SERVER> with each lsp server you"ve enabled.
 
require("lspconfig")["clangd"].setup {
	capabilities = capabilities
}

-- Theme
vim.cmd("syntax on")
vim.opt.termguicolors = true
vim.cmd("colorscheme Nordfox")
vim.cmd("set tabstop=4")
vim.cmd("set shiftwidth=4")
vim.cmd("set expandtab")
vim.cmd("set number")
require("barbar").setup{animation = false,}

-- Git
vim.g.fugitive_git_executable = "C:/Program Files/Git/bin/git.exe"

-- Clipboard
vim.opt.clipboard = "unnamedplus"

--Keymaps
require("keymaps")
```

하지만 Neovim의 경우엔 dap를 구성하는 데 좀 귀찮았다.  
특히 cpp은...

내가 쓴 dap 설정은 이렇다.

```lua
--dapconfig.lua
local M = {}

function M.setup()
	--Debug
	local dap = require("dap")
	dap.adapters.cppdbg = {
		id = "cppdbg",
		type = "executable",
		command = "path/to/vscode-cpptools/debugAdapters/bin/OpenDebugAD7.exe",
		options = {
			detached = false
		},
	}
	dap.configurations.cpp = {
		{
			id = "Launch",
			type = "cppdbg",
			request = "launch",
			program = "${fileDirname}/${fileBasenameNoExtension}.exe",
			cwd = "${fileDirname}",
			stopAtEntry = false,
			MIMode = "gdb",
			MIDebuggerPath = "path/to/msys64/mingw64/bin/gdb.exe",
			setupCommands = {  
				{	 
					text = '-enable-pretty-printing',
					description =  'enable pretty printing',
					ignoreFailures = false 
				},
			},
		},
	}
end

return M
```

(물론 경로는 알아서 잘 설정해야 한다.)

키 설정은 다음과 같이 했다.

```lua
--keymaps.lua
-- Font
local fontsize = 12
function adjust_fontsize(amount)
  fontsize = fontsize + amount
  if fontsize < 6 then
    fontsize = 6
  end
  vim.cmd('set guifont=FuraMono\\ NFM:h' .. fontsize)
end
vim.cmd('set guifont=FuraMono\\ NFM:h12')
vim.api.nvim_set_keymap('n', '<C-ScrollWheelUp>', ':lua adjust_fontsize(1)<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<C-ScrollWheelDown>', ':lua adjust_fontsize(-1)<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-ScrollWheelUp>', '<Esc>:lua adjust_fontsize(1)<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-ScrollWheelDown>', '<Esc>:lua adjust_fontsize(-1)<CR>', { noremap = true, silent = true })

-- CP Helper
vim.api.nvim_set_keymap('n', '<leader>cp', ':CompetiTestReceive problem<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>cc', ':CompetiTestReceive contest<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>cr', ':CompetiTestRun<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>cn', ':CompetiTestRunNE<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>ca', ':CompetiTestAdd<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>ce', ':CompetiTestEdit<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>cd', ':CompetiTestDelete<CR>', { noremap = true, silent = true })

-- DAP
vim.api.nvim_set_keymap('n', '<leader>b', ':w<CR> :!g++ -g -o %:r %<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>d', ':lua require("dapui").toggle()<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<F5>', ':lua require("dap").continue()<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<F10>', ':lua require("dap").step_over()<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<F11>', ':lua require("dap").step_into()<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<F12>', ':lua require("dap").step_out()<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<F9>', ':lua require("dap").toggle_breakpoint()<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader><F9>', ':lua require("dap").set_breakpoint(vim.fn.input("Breakpoint condition: "))<CR>', {silent = true})

-- ShortCuts
vim.api.nvim_set_keymap('n', '<leader>f', ':Telescope find_files<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<C-s>', ':w<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-s>', '<Esc>:w<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<C-q>', ':q<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-q>', '<Esc>:q<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<C-z>', ':u<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-z>', '<Esc>:u<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<C-y>', ':red<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-y>', '<Esc>:red<CR>', { noremap = true, silent = true })
```

Ctrl + 마우스 휠 을 이용하여 글자 크기 조절을 하고 싶어서 관련 함수를 추가했고,  
CP를 위한 Conpetitest 플러그인과 nvim-dap 관련 숏컷들을 등록해뒀다.

### 여담
저 설정들을 처음엔 Chat GPT에게 물어물어 하려고 생각했지만, **그 녀석은 생각보다 멍청했다!**  
단 하나도 진짜 도움이 될 만한 설정을 내뱉어주지 않아서, 결국 발품을 엄청 팔아서 최종적인 세팅이 완료되었다.

어쨌든, 이렇게 난 VS를 지워버릴 수 있도록 텍스트 에디터 기반의 개발 환경을 구축할 수 있었다.
